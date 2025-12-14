# Rust 和 C++ 互操作性（FFI）

> 原文：[`cel.cs.brown.edu/crp/idioms/ffi.html`](https://cel.cs.brown.edu/crp/idioms/ffi.html)

Rustonomicon [包含一个章节](https://doc.rust-lang.org/nomicon/ffi.html)，涵盖了想要从 Rust 调用 C（或通过 `extern "C"` 函数调用 C++）的 C++ 程序员或从 C 或 C++ 调用 Rust 的 Rust 程序员相关的许多问题。

许多 C 库都存在现有的 crate，既有低级绑定，也有高级安全的 Rust 抽象。例如，对于 libgit2 库，既有低级的 [libgit2-sys crate](https://crates.io/crates/libgit2-sys)，也有高级的 [git2 crate](https://crates.io/crates/git2)。

可以使用 `bindgen` 从 C 头文件生成库的绑定（[bindgen](https://rust-lang.github.io/rust-bindgen/)）。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Rust and C++ interoperability (FFI))
