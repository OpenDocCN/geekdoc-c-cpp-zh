# 06. 事件循环实现

本章将介绍一个 echo 服务器的真实 C++代码。

`struct Conn`的定义：

```cpp
enum {
    STATE_REQ = 0,
    STATE_RES = 1,
    STATE_END = 2,  // mark the connection for deletion
};

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
};
```

我们需要读写缓冲区，因为在非阻塞模式下，I/O 操作通常会被延迟。

`state`用于决定如何处理连接。对于正在进行的连接，有两个状态。`STATE_REQ`用于读取请求，而`STATE_RES`用于发送响应。

事件循环的代码：

```cpp
int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    // bind, listen and etc
    // code omitted...

    // a map of all client connections, keyed by fd
    std::vector<Conn *> fd2conn;

    // set the listen fd to nonblocking mode
    fd_set_nb(fd);

    // the event loop
    std::vector<struct pollfd> poll_args;
    while (true) {
        // prepare the arguments of the poll()
        poll_args.clear();
        // for convenience, the listening fd is put in the first position
        struct pollfd pfd = {fd, POLLIN, 0};
        poll_args.push_back(pfd);
        // connection fds
        for (Conn *conn : fd2conn) {
            if (!conn) {
                continue;
            }
            struct pollfd pfd = {};
            pfd.fd = conn->fd;
            pfd.events = (conn->state == STATE_REQ) ? POLLIN : POLLOUT;
            pfd.events = pfd.events | POLLERR;
            poll_args.push_back(pfd);
        }

        // poll for active fds
        // the timeout argument doesn't matter here
        int rv = poll(poll_args.data(), (nfds_t)poll_args.size(), 1000);
        if (rv < 0) {
            die("poll");
        }

        // process active connections
        for (size_t i = 1; i < poll_args.size(); ++i) {
            if (poll_args[i].revents) {
                Conn *conn = fd2conn[poll_args[i].fd];
                connection_io(conn);
                if (conn->state == STATE_END) {
                    // client closed normally, or something bad happened.
                    // destroy this connection
                    fd2conn[conn->fd] = NULL;
                    (void)close(conn->fd);
                    free(conn);
                }
            }
        }

        // try to accept a new connection if the listening fd is active
        if (poll_args[0].revents) {
            (void)accept_new_conn(fd2conn, fd);
        }
    }

    return 0;
}
```

在我们的事件循环中，第一件事是设置`poll`的参数。监听文件描述符（fd）使用`POLLIN`标志进行轮询。对于连接文件描述符，`Conn`结构体的状态决定了轮询标志。在这种情况下，轮询标志是读取（`POLLIN`）或写入（`POLLOUT`），永远不会同时发生。如果使用`epoll`，事件循环中的第一件事通常是使用`epoll_ctl`更新文件描述符集合。

`poll`还接受一个超时参数，可以用来实现定时器，在我们的情况下，这个参数并不重要，只需将其设置为一个很大的数字。在`poll`返回后，我们会通过哪个 fd 准备好读取/写入来通知，并相应地采取行动。

`accept_new_conn()`函数接受一个新的连接并创建`struct Conn`对象：

```cpp
static void conn_put(std::vector<Conn *> &fd2conn, struct Conn *conn) {
    if (fd2conn.size() <= (size_t)conn->fd) {
        fd2conn.resize(conn->fd + 1);
    }
    fd2conn[conn->fd] = conn;
}

static int32_t accept_new_conn(std::vector<Conn *> &fd2conn, int fd) {
    // accept
    struct sockaddr_in client_addr = {};
    socklen_t socklen = sizeof(client_addr);
    int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
    if (connfd < 0) {
        msg("accept() error");
        return -1;  // error
    }

    // set the new connection fd to nonblocking mode
    fd_set_nb(connfd);
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
    conn_put(fd2conn, conn);
    return 0;
}
```

`connection_io()`是客户端连接的状态机：

```cpp
static void connection_io(Conn *conn) {
    if (conn->state == STATE_REQ) {
        state_req(conn);
    } else if (conn->state == STATE_RES) {
        state_res(conn);
    } else {
        assert(0);  // not expected
    }
}
```

`STATE_REQ`状态是用于读取的：

```cpp
static void state_req(Conn *conn) {
    while (try_fill_buffer(conn)) {}
}

static bool try_fill_buffer(Conn *conn) {
    // try to fill the buffer
    assert(conn->rbuf_size < sizeof(conn->rbuf));
    ssize_t rv = 0;
    do {
        size_t cap = sizeof(conn->rbuf) - conn->rbuf_size;
        rv = read(conn->fd, &conn->rbuf[conn->rbuf_size], cap);
    } while (rv < 0 && errno == EINTR);
    if (rv < 0 && errno == EAGAIN) {
        // got EAGAIN, stop.
        return false;
    }
    if (rv < 0) {
        msg("read() error");
        conn->state = STATE_END;
        return false;
    }
    if (rv == 0) {
        if (conn->rbuf_size > 0) {
            msg("unexpected EOF");
        } else {
            msg("EOF");
        }
        conn->state = STATE_END;
        return false;
    }

    conn->rbuf_size += (size_t)rv;
    assert(conn->rbuf_size <= sizeof(conn->rbuf) - conn->rbuf_size);

    // Try to process requests one by one.
    // Why is there a loop? Please read the explanation of "pipelining".
    while (try_one_request(conn)) {}
    return (conn->state == STATE_REQ);
}
```

这里有很多东西需要解释。为了理解这个函数，让我们回顾一下上一章的伪代码：

```cpp
def do_something_to_client(fd):
    if should_read_from(fd):
        data = read_until_EAGAIN(fd)
        process_incoming_data(data)
    # code omitted...
```

`try_fill_buffer()`函数将数据填充到读取缓冲区中。由于读取缓冲区的大小有限，读取缓冲区在达到`EAGAIN`之前可能会填满，因此我们需要在读取后立即处理数据以清除一些读取缓冲区空间，然后`try_fill_buffer()`会循环直到达到`EAGAIN`。

在得到`errno EINTR`后，需要重试`read`系统调用（以及任何其他系统调用）。`EINTR`表示系统调用被信号中断，即使我们的应用程序没有使用信号，也需要重试。

`try_one_request`函数处理传入的数据，但为什么这个函数是循环的？读取缓冲区中是否只有一个请求？答案是肯定的。对于请求/响应协议，客户端不是一次只能发送一个请求并等待响应，客户端可以通过在不等待响应之间发送多个请求来节省一些延迟，这种操作模式称为“流水线”。因此，我们不能假设读取缓冲区中最多只有一个请求。

列出`try_one_request`函数：

```cpp
static bool try_one_request(Conn *conn) {
    // try to parse a request from the buffer
    if (conn->rbuf_size < 4) {
        // not enough data in the buffer. Will retry in the next iteration
        return false;
    }
    uint32_t len = 0;
    memcpy(&len, &conn->rbuf[0], 4);
    if (len > k_max_msg) {
        msg("too long");
        conn->state = STATE_END;
        return false;
    }
    if (4 + len > conn->rbuf_size) {
        // not enough data in the buffer. Will retry in the next iteration
        return false;
    }

    // got one request, do something with it
    printf("client says: %.*s\n", len, &conn->rbuf[4]);

    // generating echoing response
    memcpy(&conn->wbuf[0], &len, 4);
    memcpy(&conn->wbuf[4], &conn->rbuf[4], len);
    conn->wbuf_size = 4 + len;

    // remove the request from the buffer.
    // note: frequent memmove is inefficient.
    // note: need better handling for production code.
    size_t remain = conn->rbuf_size - 4 - len;
    if (remain) {
        memmove(conn->rbuf, &conn->rbuf[4 + len], remain);
    }
    conn->rbuf_size = remain;

    // change state
    conn->state = STATE_RES;
    state_res(conn);

    // continue the outer loop if the request was fully processed
    return (conn->state == STATE_REQ);
}
```

`try_one_request`函数从一个读取缓冲区中取一个请求，生成一个响应，然后转换到`STATE_RES`状态。

`STATE_RES` 状态的代码：

```cpp
static void state_res(Conn *conn) {
    while (try_flush_buffer(conn)) {}
}

static bool try_flush_buffer(Conn *conn) {
    ssize_t rv = 0;
    do {
        size_t remain = conn->wbuf_size - conn->wbuf_sent;
        rv = write(conn->fd, &conn->wbuf[conn->wbuf_sent], remain);
    if (rv < 0 && errno == EAGAIN) {
        // got EAGAIN, stop.
        return false;
    }
    if (rv < 0) {
        msg("write() error");
        conn->state = STATE_END;
        return false;
    }
    conn->wbuf_sent += (size_t)rv;
    assert(conn->wbuf_sent <= conn->wbuf_size);
    if (conn->wbuf_sent == conn->wbuf_size) {
        // response was fully sent, change state back
        conn->state = STATE_REQ;
        conn->wbuf_sent = 0;
        conn->wbuf_size = 0;
        return false;
    }
    // still got some data in wbuf, could try to write again
    return true;
}
```

上述代码将写入缓冲区刷新直到得到`EAGAIN`，或者如果刷新完成，则转换回`STATE_REQ`状态。

要测试我们的服务器，我们可以运行第四章中的客户端，因为协议是相同的。我们也可以修改客户端来演示流水线客户端：

```cpp
// the `query` function was simply splited into `send_req` and `read_res`.
static int32_t send_req(int fd, const char *text);
static int32_t read_res(int fd);

int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    // code omitted...

    // multiple pipelined requests
    const char *query_list[3] = {"hello1", "hello2", "hello3"};
    for (size_t i = 0; i < 3; ++i) {
        int32_t err = send_req(fd, query_list[i]);
        if (err) {
            goto L_DONE;
        }
    }
    for (size_t i = 0; i < 3; ++i) {
        int32_t err = read_res(fd);
        if (err) {
            goto L_DONE;
        }
    }

L_DONE:
    close(fd);
    return 0;
}
```

练习：

1.  尝试在事件循环中使用 `epoll` 而不是 `poll`。这应该很容易。

1.  我们正在使用 `memmove` 来回收读取缓冲区的空间。然而，在每次请求上使用 `memmove` 是不必要的，只需在 `read` 之前执行 `memmove` 的代码即可。

1.  在 `state_res` 函数中，对单个响应执行了 `write` 操作。在流水线场景中，我们可以缓冲多个响应，并在最后通过单个 `write` 调用来刷新它们。请注意，写入缓冲区可能在中间已满。

> +   [06_client.cpp](https://build-your-own.org/redis/06/06_client.cpp.htm)
> +   
> +   [06_server.cpp](https://build-your-own.org/redis/06/06_server.cpp.htm)
