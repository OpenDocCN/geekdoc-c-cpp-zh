# 文档（例如，Doxygen）

> 原文：[`cel.cs.brown.edu/crp/etc/documentation.html`](https://cel.cs.brown.edu/crp/etc/documentation.html)

虽然 C++ 有几个文档工具，如 Doxygen 和 Sphinx，但 Rust 只有一个文档工具，即[Rustdoc](https://doc.rust-lang.org/rustdoc/index.html)。Rustdoc 由`docs.rs`、cargo、[rust-analyzer](https://rust-analyzer.github.io/)支持，并用于记录[标准库](https://doc.rust-lang.org/std/vec/struct.Vec.html#blanket-implementations)。对于大多数发行版，Rustdoc 默认与 Rust 工具链一起安装。

Rustdoc 可用的功能和选项在[Rustdoc 手册](https://doc.rust-lang.org/rustdoc/)中有记录。该手册还记录了[记录 Rust 代码的最佳实践](https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html)，这些最佳实践与使用 Doxygen 记录 C++代码的推荐实践略有不同。

Cargo 集成在 Cargo 手册的`cargo doc`和`cargo rustdoc`命令下进行文档记录，以及`cargo test`命令的[doctests 部分](https://doc.rust-lang.org/cargo/commands/cargo-test.html#documentation-tests)。

本章比较了 Rustdoc 与 Doxygen 的一些方面，以帮助理解从 Doxygen 或类似的 C++文档工具迁移到 Rustdoc 时可以期待什么。

## 输出格式

与 Doxygen 不同，Doxygen 还可以生成 PDF 和 man 页面输出，Rustdoc 只生成 HTML 输出。生成的文档包括客户端搜索功能，这包括通过类型签名进行搜索的能力。

[rust-analyzer](https://rust-analyzer.github.io/)语言服务器支持 Rustdoc 注释，并通过语言服务器协议支持在悬停时将它们提供给具有语言服务器协议支持的编辑器，即使是当前项目的 Rustdoc 注释。

## Rustdoc 注释语法

与支持多种 C++注释语法的 Doxygen 不同，Rustup 只支持一种注释语法。以`//!`开头的注释用于记录顶层模块或 crate。以`///`开头的注释用于记录后续的项目。

```rs
/**
 * @file myheader.h
 * @brief A description of this file.
 *
 * A longer description, with examples, etc.
 */

/**
 * @brief A description of this class.
 *
 * A longr description, with examples, etc.
 */
struct MyClass {
  // ...
}; 
```

```rs

#![allow(unused)] fn main() { //! 对此模块或 crate 的描述。

//!

//! 更长的描述，包括示例等。

/// 对此类型的描述。

///

/// 更长的描述，包括示例等。

struct MyClass {

    // ...

}

}

```

### 特殊形式

注释内容直到第一行空白与 Doxygen 中的`@brief`形式处理类似。

除了这些，Rustdoc 没有专门的形式来记录项的各个部分，例如函数的参数。相反，可以使用[Markdown 语法](https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html#markdown)来格式化文档，否则文档将以散文形式给出。

有几种常见的约定用于结构化文档注释。最常见的约定是包括以下必要的部分（使用 Markdown 标题语法定义）：

+   panics（对于会 panic 的函数，例如在[`Vec::split_at`](https://doc.rust-lang.org/std/vec/struct.Vec.html#panics-32)上），

+   safety（对于不安全的函数，例如在[`Vec::split_at_unchecked`](https://doc.rust-lang.org/std/primitive.slice.html#safety-10)上），以及

+   示例（例如，在[`Vec::split_at`](https://doc.rust-lang.org/std/vec/struct.Vec.html#examples-105)上）。

以下注释比较了使用 Doxygen 记录的 C++函数的文档和使用 Rustdoc 记录的 Rust 函数的文档。

```rs
/**
 * @brief Computes the factorial.
 *
 * Computes the factorial in a stack-safe way.
 * The factorial is defined as...
 *
 * @code
 * #include <cassert>
 * #include "factorial.h"
 *
 * int main() {
 *    int res = factorial(3);
 *    assert(6 == res);
 * }
 * @endcode
 *
 * @param n The number of which to take the factorial
 *
 * @return The factorial
 *
 * @exception domain_error If n < 0
 */
int factorial(int n); 
```

```rs

#![allow(unused)] fn main() { /// 计算阶乘。

///

/// 以堆栈安全的方式计算阶乘。

/// 阶乘被定义为...

///

/// # 示例

///

/// ```rs
/// let res = factorial(3);
/// assert_eq!(6, res);
/// ```

///

/// # Panics

///

/// 要求`n >= 0`，否则会 panic。

/// 对于非 panic 版本，请参阅

/// [`factorial_checked`].

fn factorial(n: i32) -> i32 {

    // ...

todo!() }

}

```

### 自动文档

许多可以从代码中推导出的内容都会被 Rustdoc 自动包含。其中之一是，对于类型（例如，在[`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html#trait-implementations)上的）的特性和泛型实现（例如，在[`Vec`](https://doc.rust-lang.org/std/vec/struct.Vec.html#blanket-implementations)上的）不需要手动文档化，因为 Rustdoc 会自动发现并包含这些在 crate 中可见的实现。

# 附加功能

一些有价值的 Rustdoc 特性可能不会让从使用 Doxygen 的人预料到。因为这些特性提供了显著的好处，所以在这里指出。

## 通过 Rustdoc 和`cargo test`支持 Doctest

当使用 Rustdoc 记录 Rust 程序时，包括示例的一个具体好处是，[在运行`cargo test`时，这些示例可以包含在测试套件中](https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html)。

Rustdoc 处理示例作为测试的逻辑包括处理部分程序，因此即使是早期比较中的代码示例也可以作为测试使用。

## 项目及安装库的本地文档

可以使用 `cargo doc --open` 在浏览器中查看工作项目和依赖库的本地文档。通过使用 `cargo doc --open --document-private-items` 可以将项目的私有项包含在文档中。因为 Rustdoc 注释也被 [rust-analyzer](https://rust-analyzer.github.io/) 语言服务器用于在兼容的编辑器中提供悬停文档，所以使用 Rustdoc 注释来记录私有项通常是值得的。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Documentation (e.g., Doxygen)) [](../etc/tests.html "上一章") *[](../etc/build_systems.html "下一章")*** [](../etc/tests.html "上一章")* [](../etc/build_systems.html "下一章")***
