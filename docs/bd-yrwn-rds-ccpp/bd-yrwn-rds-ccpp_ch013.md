# 12. 事件循环和计时器

我们的服务器中缺少一个主要的东西：超时。每个网络应用程序都需要处理超时，因为网络另一端可能会消失。不仅进行中的 IO 操作如读写需要超时，而且踢出空闲的 TCP 连接也是一个好主意。要实现超时，必须修改事件循环，因为`poll`是唯一会阻塞的操作。

看看我们现有的事件循环代码：

```cpp
int rv = poll(poll_args.data(), (nfds_t)poll_args.size(), 1000);
```

`poll`系统调用接受一个超时参数，它对`poll`系统调用所花费的时间施加了一个上限。当前的超时值是一个任意的 1000 毫秒值。如果我们根据计时器设置超时值，`poll`应该在超时时间到达时或之前唤醒；这样我们就有机会及时触发计时器。

问题是我们可能有多个计时器，`poll`的超时值应该是最近计时器的超时值。需要某种数据结构来找到最近的计时器。堆数据结构是寻找最小/最大值的一个流行选择，并且经常用于此类目的。也可以使用任何排序数据结构。例如，我们可以使用 AVL 树来排序计时器，并可能增强树以跟踪最小值。

让我们先添加计时器来踢出空闲的 TCP 连接。对于每个连接都有一个计时器，设置为未来的固定超时时间，每次在连接上有 IO 活动时，计时器都会更新为固定超时时间。注意，当我们更新计时器时，它变成了最远的那个；因此，我们可以利用这个事实来简化数据结构；一个简单的链表就足以保持计时器的顺序：新的或更新的计时器简单地移动到列表的末尾，而列表保持排序顺序。此外，链表操作的时间复杂度是`O(1)`，这比排序数据结构要好。

定义链表是一个简单任务：

```cpp
struct DList {
    DList *prev = NULL;
    DList *next = NULL;
};

inline void dlist_init(DList *node) {
    node->prev = node->next = node;
}

inline bool dlist_empty(DList *node) {
    return node->next == node;
}

inline void dlist_detach(DList *node) {
    DList *prev = node->prev;
    DList *next = node->next;
    prev->next = next;
    next->prev = prev;
}

inline void dlist_insert_before(DList *target, DList *rookie) {
    DList *prev = target->prev;
    prev->next = rookie;
    rookie->prev = prev;
    rookie->next = target;
    target->prev = rookie;
}
```

`get_monotonic_usec`是获取时间的函数。注意，时间戳必须是单调的。时间戳回跳可能会在计算机系统中引起各种问题。

```cpp
static uint64_t get_monotonic_usec() {
    timespec tv = {0, 0};
    clock_gettime(CLOCK_MONOTONIC, &tv);
    return uint64_t(tv.tv_sec) * 1000000 + tv.tv_nsec / 1000;
}
```

下一步是将列表添加到服务器和连接结构体中。

```cpp
// global variables
static struct {
    HMap db;
    // a map of all client connections, keyed by fd
    std::vector<Conn *> fd2conn;
    // timers for idle connections
    DList idle_list;
} g_data;
```

```cpp
struct Conn {
    int fd = -1;
    uint32_t state = 0;     // either STATE_REQ or STATE_RES
    // buffer for reading
    size_t rbuf_size = 0;
    uint8_t rbuf[4 + k_max_msg];
    // buffer for writing
    size_t wbuf_size = 0;
    size_t wbuf_sent = 0;
    uint8_t wbuf[4 + k_max_msg];
    uint64_t idle_start = 0;
    // timer
    DList idle_list;
};
```

修改后的事件循环概述：

```cpp
int main() {
    // some initializations
    dlist_init(&g_data.idle_list);

    int fd = socket(AF_INET, SOCK_STREAM, 0);
    // bind, listen & other miscs
    // code omitted...

    // the event loop
    std::vector<struct pollfd> poll_args;
    while (true) {
        // prepare the arguments of the poll()
        // code omitted...

        // poll for active fds
        int timeout_ms = (int)next_timer_ms();
        int rv = poll(poll_args.data(), (nfds_t)poll_args.size(), timeout_ms);
        if (rv < 0) {
            die("poll");
        }

        // process active connections
        for (size_t i = 1; i < poll_args.size(); ++i) {
            if (poll_args[i].revents) {
                Conn *conn = g_data.fd2conn[poll_args[i].fd];
                connection_io(conn);
                if (conn->state == STATE_END) {
                    // client closed normally, or something bad happened.
                    // destroy this connection
                    conn_done(conn);
                }
            }
        }

        // handle timers
        process_timers();

        // try to accept a new connection if the listening fd is active
        if (poll_args[0].revents) {
            (void)accept_new_conn(fd);
        }
    }

    return 0;
}
```

修改了几件事：

1.  `poll`的超时参数是由`next_timer_ms`函数计算的。

1.  销毁连接的代码被移动到了`conn_done`函数中。

1.  添加了`process_timers`函数来触发计时器。

1.  计时器在`connection_io`中更新，并在`accept_new_conn`中初始化。

`next_timer_ms`函数从列表中取出第一个（最近的）计时器，并使用它来计算`poll`的超时值。

```cpp
const uint64_t k_idle_timeout_ms = 5 * 1000;

static uint32_t next_timer_ms() {
    if (dlist_empty(&g_data.idle_list)) {
        return 10000;   // no timer, the value doesn't matter
    }

    uint64_t now_us = get_monotonic_usec();
    Conn *next = container_of(g_data.idle_list.next, Conn, idle_list);
    uint64_t next_us = next->idle_start + k_idle_timeout_ms * 1000;
    if (next_us <= now_us) {
        // missed?
        return 0;
    }

    return (uint32_t)((next_us - now_us) / 1000);
}
```

在事件循环的每次迭代中，都会检查列表以及时触发计时器。

```cpp
static void process_timers() {
    uint64_t now_us = get_monotonic_usec();
    while (!dlist_empty(&g_data.idle_list)) {
        Conn *next = container_of(g_data.idle_list.next, Conn, idle_list);
        uint64_t next_us = next->idle_start + k_idle_timeout_ms * 1000;
        if (next_us >= now_us + 1000) {
            // not ready, the extra 1000us is for the ms resolution of poll()
            break;
        }

        printf("removing idle connection: %d\n", next->fd);
        conn_done(next);
    }
}
```

计时器在`connection_io`函数中更新：

```cpp
static void connection_io(Conn *conn) {
    // waked up by poll, update the idle timer
    // by moving conn to the end of the list.
    conn->idle_start = get_monotonic_usec();
    dlist_detach(&conn->idle_list);
    dlist_insert_before(&g_data.idle_list, &conn->idle_list);

    // do the work
    if (conn->state == STATE_REQ) {
        state_req(conn);
    } else if (conn->state == STATE_RES) {
        state_res(conn);
    } else {
        assert(0);  // not expected
    }
}
```

计时器在`accept_new_conn`函数中初始化：

```cpp
static int32_t accept_new_conn(int fd) {
    // code omitted...

    // creating the struct Conn
    struct Conn *conn = (struct Conn *)malloc(sizeof(struct Conn));
    if (!conn) {
        close(connfd);
        return -1;
    }
    conn->fd = connfd;
    conn->state = STATE_REQ;
    conn->rbuf_size = 0;
    conn->wbuf_size = 0;
    conn->wbuf_sent = 0;
    conn->idle_start = get_monotonic_usec();
    dlist_insert_before(&g_data.idle_list, &conn->idle_list);
    conn_put(g_data.fd2conn, conn);
    return 0;
}
```

完成后不要忘记从列表中移除连接：

```cpp
static void conn_done(Conn *conn) {
    g_data.fd2conn[conn->fd] = NULL;
    (void)close(conn->fd);
    dlist_detach(&conn->idle_list);
    free(conn);
}
```

我们可以使用`nc`或`socat`命令测试空闲超时：

```cpp
$ ./server
removing idle connection: 4
```

```cpp
$ socat tcp:127.0.0.1:1234 -
```

服务器应在 5 秒内关闭连接。

练习：

1.  为 IO 操作（读取和写入）添加超时。

1.  尝试使用排序数据结构实现更通用的计时器。

> +   [12_server.cpp](https://build-your-own.org/redis/12/12_server.cpp.htm)
> +   
> +   [avl.cpp](https://build-your-own.org/redis/12/avl.cpp.htm)
> +   
> +   [avl.h](https://build-your-own.org/redis/12/avl.h.htm)
> +   
> +   [common.h](https://build-your-own.org/redis/12/common.h.htm)
> +   
> +   [hashtable.cpp](https://build-your-own.org/redis/12/hashtable.cpp.htm)
> +   
> +   [hashtable.h](https://build-your-own.org/redis/12/hashtable.h.htm)
> +   
> +   [list.h](https://build-your-own.org/redis/12/list.h.htm)
> +   
> +   [zset.cpp](https://build-your-own.org/redis/12/zset.cpp.htm)
> +   
> +   [zset.h](https://build-your-own.org/redis/12/zset.h.htm)
