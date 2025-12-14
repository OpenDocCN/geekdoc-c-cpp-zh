# 预分配缓冲区

> 原文：[`cel.cs.brown.edu/crp/idioms/out_params/pre-allocated_buffers.html`](https://cel.cs.brown.edu/crp/idioms/out_params/pre-allocated_buffers.html)

有一些情况需要从将被反复调用的函数中返回大量数据，因此涉及通过值返回或重复堆分配的副本将会成本高昂。这些情况包括：

+   执行文件或网络 IO，

+   与图形硬件通信，

+   与嵌入式系统上的硬件通信，或

+   实现加密算法。

在这种情况下，C++程序往往会预分配缓冲区，这些缓冲区将被用于所有调用。这通常也使得可以在栈上分配缓冲区，而不是必须使用动态存储。

以下示例在循环中预分配一个缓冲区并将大文件读入其中。

```rs
#include <fstream>

int main() {
  std::ifstream file("/path/to/file");
  if (!file.is_open()) {
    return -1;
  }

  byte buf[1024];
  while (file.good()) {
    file.read(buf, sizeof buf);
    std::streamsize count = file.gcount();

    // use data in buf
  }

  return 0;
} 
```

```rs

```

use std::fs::File;

use std::io::{BufReader, Read};

fn main() -> Result<(), std::io::Error> {

    let mut f = BufReader::new(File::open(

        "/path/to/file",

    )?);

    let mut buf = [0u8; 1024];

    loop {

        let count = f.read(&mut buf)?;

        if count == 0 {

            break;

        }

        // 使用 buf 中的数据

    }

    Ok(())

}

```rs

```

C++程序和 Rust 程序之间的主要区别在于，在 Rust 程序中，必须在使用之前初始化缓冲区。在大多数情况下，这种一次性初始化成本并不显著。当它是的时候，需要使用不安全 Rust 来避免初始化。

避免初始化的技术使用了`[`std::mem::MaybeUninit`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html)。API 文档中给出了该类型的[安全使用`MaybeUninit`的示例](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#examples)。

稳定版 Rust 的 IO API 不包含对`MaybeUninit`的支持。相反，正在开发一个新的安全 API，它将允许在不要求使用不安全 Rust 的情况下避免初始化。 [关于`MaybeUninit`的安全使用示例](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html)可以在该类型的 API 文档中找到。

如果被调用者可能需要扩展提供的缓冲区并且允许动态分配，则可以使用`&mut Vec<T>`而不是`&mut [T]`。这类似于在 C++中提供`std::vector<T>&`。为了避免不必要的重新分配，可以使用`Vec::<T>::with_capacity(n)`来创建向量。

## 关于读取文件的说明

虽然这里的示例使用 IO 来演示重用预分配的缓冲区，但还有更高层次的接口可以从`File`中读取，无论是从`[`Read`](https://doc.rust-lang.org/std/io/trait.Read.html)`和`[`BufRead`](https://doc.rust-lang.org/std/io/trait.BufRead.html)`特质，还是从`[`std::io`](https://doc.rust-lang.org/std/io/index.html#functions-1)`和`[`std::fs`](https://doc.rust-lang.org/std/fs/index.html#functions-1)`中的便利函数。

这里描述的技术在其他需要可重用缓冲区的情况下很有用，例如在与硬件 API 交互时，在使用现有的 C 或 C++库时，或者当实现产生大量数据的算法时，例如加密算法。

## 即将到来的更改和 `BorrowedBuf`

Rust 社区正在改进处理未初始化缓冲区的方法。在 Rust 的 nightly 分支上，可以使用[`BorrowedBuf`](https://doc.rust-lang.org/std/io/struct.BorrowedBuf.html)来实现与使用`MaybeUninit`切片相同的结果，但无需编写任何不安全代码。用于避免不必要的初始化的 IO API 使用`BorrowedBuf`而不是`MaybeUninit`切片。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Pre-allocated buffers)
