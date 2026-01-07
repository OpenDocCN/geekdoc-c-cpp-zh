# 03\. Hello Server/Client

本章继续介绍套接字编程。我们将编写两个简单的（不完整且损坏的）程序来演示上一章的 syscalls。第一个程序是一个服务器，它接受来自客户端的连接，读取一条消息，并写入一条回复。第二个程序是一个客户端，它连接到服务器，写入一条消息，并读取一条回复。我们先从服务器开始。

首先，我们需要获取一个套接字文件描述符：`int fd = socket(AF_INET, SOCK_STREAM, 0);`

`AF_INET` 用于 IPv4，使用 `AF_INET6` 用于 IPv6 或双栈套接字。为了简单起见，本书中我们将只使用 `AF_INET`。

`SOCK_STREAM` 用于 TCP。本书中我们不会使用除 TCP 之外的其他任何东西。`socket()` 调用的所有 3 个参数在本书中都是固定的。

接下来，我们将介绍一个新的系统调用：

```cpp
int val = 1;
setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &val, sizeof(val));
```

`setsockopt()` 调用用于配置套接字的各个方面。这个特定的调用启用了 `SO_REUSEADDR` 选项。如果没有这个选项，服务器在重启后无法绑定到相同的地址。读者练习：找出 `SO_REUSEADDR` 的确切含义以及为什么需要它。

下一步是 `bind()` 和 `listen()`，我们将绑定到通配符地址 `0.0.0.0:1234`：

```cpp
    // bind, this is the syntax that deals with IPv4 addresses
    struct sockaddr_in addr = {};
    addr.sin_family = AF_INET;
    addr.sin_port = ntohs(1234);
    addr.sin_addr.s_addr = ntohl(0);    // wildcard address 0.0.0.0
    int rv = bind(fd, (const sockaddr *)&addr, sizeof(addr));
    if (rv) {
        die("bind()");
    }

    // listen
    rv = listen(fd, SOMAXCONN);
    if (rv) {
        die("listen()");
    }
```

遍历每个连接并对它们进行操作。

```cpp
    while (true) {
        // accept
        struct sockaddr_in client_addr = {};
        socklen_t socklen = sizeof(client_addr);
        int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
        if (connfd < 0) {
            continue;   // error
        }

        do_something(connfd);
        close(connfd);
    }
```

`do_something()` 函数只是读取和写入。

```cpp
static void do_something(int connfd) {
    char rbuf[64] = {};
    ssize_t n = read(connfd, rbuf, sizeof(rbuf) - 1);
    if (n < 0) {
        msg("read() error");
        return;
    }
    printf("client says: %s\n", rbuf);

    char wbuf[] = "world";
    write(connfd, wbuf, strlen(wbuf));
}
```

注意，`read()` 和 `write()` 调用返回读取或写入的字节数。真正的程序员必须处理函数的返回值，但为了简洁，本章中我省略了很多内容。而且，本章中的代码根本不是进行网络通信的正确方式。

客户端程序要简单得多：

```cpp
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    struct sockaddr_in addr = {};
    addr.sin_family = AF_INET;
    addr.sin_port = ntohs(1234);
    addr.sin_addr.s_addr = ntohl(INADDR_LOOPBACK);  // 127.0.0.1
    int rv = connect(fd, (const struct sockaddr *)&addr, sizeof(addr));
    if (rv) {
        die("connect");
    }

    char msg[] = "hello";
    write(fd, msg, strlen(msg));

    char rbuf[64] = {};
    ssize_t n = read(fd, rbuf, sizeof(rbuf) - 1);
    if (n < 0) {
        die("read");
    }
    printf("server says: %s\n", rbuf);
    close(fd);
```

使用以下命令行编译我们的程序：

```cpp
g++ -Wall -Wextra -O2 -g 03_server.cpp -o server
g++ -Wall -Wextra -O2 -g 03_client.cpp -o client
```

在一个窗口中运行 `./server`，然后在另一个窗口中运行 `./client`。你应该看到以下结果：

```cpp
$ ./server
client says: hello
```

```cpp
$ ./client
server says: world
```

读者练习：阅读本章中使用的 API 的手册页，或查找相关的在线教程。确保你知道如何查找 API 的帮助，因为本书不会涵盖 API 使用的细节。

> +   [03_client.cpp](https://build-your-own.org/redis/03/03_client.cpp.htm)
> +   
> +   [03_server.cpp](https://build-your-own.org/redis/03/03_server.cpp.htm)
