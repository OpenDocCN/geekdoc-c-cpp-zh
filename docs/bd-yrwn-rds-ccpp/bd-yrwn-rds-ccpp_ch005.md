# 04. 协议解析

我们的服务器将能够处理来自客户端的多个请求，为了做到这一点，我们需要实现某种“协议”，至少要将请求与 TCP 字节流分开。将请求分开的最简单方法是在请求的开头声明请求的长度。让我们使用以下方案。

```cpp
+-----+------+-----+------+--------
| len | msg1 | len | msg2 | more...
+-----+------+-----+------+--------
```

协议由两部分组成：一个 4 字节的 Little-endian 整数，表示后续请求的长度，以及一个可变长度的请求。

从上一章的代码开始，服务器的循环被修改以处理多个请求：

```cpp
    while (true) {
        // accept
        struct sockaddr_in client_addr = {};
        socklen_t socklen = sizeof(client_addr);
        int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
        if (connfd < 0) {
            continue;   // error
        }

        // only serves one client connection at once
        while (true) {
            int32_t err = one_request(connfd);
            if (err) {
                break;
            }
        }
        close(connfd);
    }
```

`one_request` 函数只解析一个请求和回复，直到发生错误或客户端连接断开。我们的服务器只能同时处理一个连接，直到我们在后续章节中引入事件循环。

在列出 `one_request` 函数之前添加两个辅助函数：

```cpp
static int32_t read_full(int fd, char *buf, size_t n) {
    while (n > 0) {
        ssize_t rv = read(fd, buf, n);
        if (rv <= 0) {
            return -1;  // error, or unexpected EOF
        }
        assert((size_t)rv <= n);
        n -= (size_t)rv;
        buf += rv;
    }
    return 0;
}

static int32_t write_all(int fd, const char *buf, size_t n) {
    while (n > 0) {
        ssize_t rv = write(fd, buf, n);
        if (rv <= 0) {
            return -1;  // error
        }
        assert((size_t)rv <= n);
        n -= (size_t)rv;
        buf += rv;
    }
    return 0;
}
```

以下是需要注意的两点：

1.  `read()` 系统调用只返回内核中可用的任何数据，如果没有数据则阻塞。处理数据不足的责任在于应用程序。`read_full()` 函数从内核中读取，直到获取到恰好 `n` 个字节。

1.  同样，如果内核缓冲区已满，`write()` 系统调用可以返回成功并写入部分数据，我们需要在 `write()` 返回少于所需字节数时继续尝试。

`one_request` 函数执行实际工作：

```cpp
const size_t k_max_msg = 4096;

static int32_t one_request(int connfd) {
    // 4 bytes header
    char rbuf[4 + k_max_msg + 1];
    errno = 0;
    int32_t err = read_full(connfd, rbuf, 4);
    if (err) {
        if (errno == 0) {
            msg("EOF");
        } else {
            msg("read() error");
        }
        return err;
    }

    uint32_t len = 0;
    memcpy(&len, rbuf, 4);  // assume little endian
    if (len > k_max_msg) {
        msg("too long");
        return -1;
    }

    // request body
    err = read_full(connfd, &rbuf[4], len);
    if (err) {
        msg("read() error");
        return err;
    }

    // do something
    rbuf[4 + len] = '\0';
    printf("client says: %s\n", &rbuf[4]);

    // reply using the same protocol
    const char reply[] = "world";
    char wbuf[4 + sizeof(reply)];
    len = (uint32_t)strlen(reply);
    memcpy(wbuf, &len, 4);
    memcpy(&wbuf[4], reply, len);
    return write_all(connfd, wbuf, 4 + len);
}
```

为了方便起见，我们添加了最大请求大小的限制，并使用足够大的缓冲区来存储请求。在解析协议时，字节序曾经是一个考虑因素，但今天它不太相关，所以我们只是使用 `memcpy` 来复制整数。

用于发送请求和接收回复的客户端代码：

```cpp
static int32_t query(int fd, const char *text) {
    uint32_t len = (uint32_t)strlen(text);
    if (len > k_max_msg) {
        return -1;
    }

    char wbuf[4 + k_max_msg];
    memcpy(wbuf, &len, 4);  // assume little endian
    memcpy(&wbuf[4], text, len);
    if (int32_t err = write_all(fd, wbuf, 4 + len)) {
        return err;
    }

    // 4 bytes header
    char rbuf[4 + k_max_msg + 1];
    errno = 0;
    int32_t err = read_full(fd, rbuf, 4);
    if (err) {
        if (errno == 0) {
            msg("EOF");
        } else {
            msg("read() error");
        }
        return err;
    }

    memcpy(&len, rbuf, 4);  // assume little endian
    if (len > k_max_msg) {
        msg("too long");
        return -1;
    }

    // reply body
    err = read_full(fd, &rbuf[4], len);
    if (err) {
        msg("read() error");
        return err;
    }

    // do something
    rbuf[4 + len] = '\0';
    printf("server says: %s\n", &rbuf[4]);
    return 0;
}
```

通过发送多个命令来测试我们的服务器：

```cpp
int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    // code omitted ...

    // multiple requests
    int32_t err = query(fd, "hello1");
    if (err) {
        goto L_DONE;
    }
    err = query(fd, "hello2");
    if (err) {
        goto L_DONE;
    }
    err = query(fd, "hello3");
    if (err) {
        goto L_DONE;
    }

L_DONE:
    close(fd);
    return 0;
}
```

运行服务器和客户端：

```cpp
$ ./server
client says: hello1
client says: hello2
client says: hello3
EOF

$ ./client
server says: world
server says: world
server says: world
```

协议解析代码每个请求至少需要 2 次 `read()` 系统调用。可以通过使用“缓冲 I/O”来减少系统调用的次数。也就是说：一次尽可能多地读取到缓冲区中，然后尝试从该缓冲区解析多个请求。鼓励读者尝试这个练习，因为这可能有助于理解后续章节。

关于协议的说明：本章中使用的协议是最简单的实用协议。大多数现实世界的协议比这更复杂。一些使用文本而不是二进制数据。虽然文本协议具有可读性强的优势，但文本协议确实需要比二进制协议更多的解析，二进制协议更易于编码且更容易出错。使协议解析复杂化的另一个因素是，某些协议没有直接分割消息的方法，这些协议可能使用分隔符，或者需要进一步解析来分割消息。当协议携带任意数据时，使用分隔符可能会增加另一个复杂性，因为数据中的分隔符需要“转义”。我们将在后续章节中坚持使用简单的二进制协议。

> +   [04_client.cpp](https://build-your-own.org/redis/04/04_client.cpp.htm)
> +   
> +   [04_server.cpp](https://build-your-own.org/redis/04/04_server.cpp.htm)
