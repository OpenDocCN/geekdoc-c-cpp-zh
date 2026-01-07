# 05\. 事件循环和非阻塞 I/O

在服务器端网络编程中处理并发连接有三种方法。它们是：进程创建、多线程和事件循环。进程创建为每个客户端连接创建新的进程以实现并发。多线程使用线程而不是进程。事件循环使用轮询和非阻塞 I/O，通常在单个线程上运行。由于进程和线程的开销，大多数现代生产级软件使用事件循环进行网络编程。

我们服务器的简化伪代码如下：

```cpp
all_fds = [...]
while True:
    active_fds = poll(all_fds)
    for each fd in active_fds:
        do_something_with(fd)

def do_something_with(fd):
    if fd is a listening socket:
        add_new_client(fd)
    elif fd is a client connection:
        while work_not_done(fd):
            do_something_to_client(fd)

def do_something_to_client(fd):
    if should_read_from(fd):
        data = read_until_EAGAIN(fd)
        process_incoming_data(data)
    while should_write_to(fd):
        write_until_EAGAIN(fd)
    if should_close(fd):
        destroy_client(fd)
```

我们不是仅仅使用 fds（文件描述符）进行（读取、写入或接受）操作，而是使用 `poll` 操作来告诉我们哪个 fd 可以立即操作而不阻塞。当我们对一个 fd 执行 I/O 操作时，该操作应该在非阻塞模式下进行。

在阻塞模式下，当内核中没有数据时，`read` 会阻塞调用者，当写入缓冲区满时，`write` 会阻塞，当内核队列中没有新连接时，`accept` 会阻塞。在非阻塞模式下，这些操作要么不阻塞成功，要么失败并返回 errno `EAGAIN`，这意味着“未准备好”。失败并返回 `EAGAIN` 的非阻塞操作必须在 `poll` 通知就绪后重试。

`poll` 是事件循环中的唯一阻塞操作，其他所有操作都必须是非阻塞的；因此，单个线程可以处理多个并发连接。所有阻塞的网络 I/O API，如 `read`、`write` 和 `accept`，都有一个非阻塞模式。没有非阻塞模式的 API，如 `gethostbyname` 和磁盘 I/O，应该在线程池中执行，这将在后面的章节中介绍。此外，定时器必须在事件循环中实现，因为我们不能在事件循环内部 `sleep` 等待。

将 fd 设置为非阻塞模式的系统调用是 `fcntl`：

```cpp
static void fd_set_nb(int fd) {
    errno = 0;
    int flags = fcntl(fd, F_GETFL, 0);
    if (errno) {
        die("fcntl error");
        return;
    }

    flags |= O_NONBLOCK;

    errno = 0;
    (void)fcntl(fd, F_SETFL, flags);
    if (errno) {
        die("fcntl error");
    }
}
```

在 Linux 上，除了 `poll` 系统调用外，还有 `select` 和 `epoll`。古老的 `select` 系统调用基本上与 `poll` 相同，除了最大 fd 数量限制在一个较小的数字，这使得它在现代应用中变得过时。`epoll` API 由 3 个系统调用组成：`epoll_create`、`epoll_wait` 和 `epoll_ctl`。`epoll` API 是状态化的，而不是像系统调用参数一样提供一组 fd，`epoll_ctl` 用于操作由 `epoll_create` 创建的 fd 集合，而 `epoll_wait` 正在操作这个集合。

我们将在下一章中使用 `poll` 系统调用，因为它比状态化的 `epoll` API 代码略少。然而，在现实世界的项目中，`epoll` API 更受欢迎，因为随着文件描述符数量的增加，`poll` 的参数可能会变得过大。
