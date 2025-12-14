# 空值 (nullptr)

> 原文：[`cel.cs.brown.edu/crp/idioms/null.html`](https://cel.cs.brown.edu/crp/idioms/null.html)

本节涵盖了 C++中`nullptr`的惯用用法以及如何在 Rust 中实现相同的结果。

由于其他语言差异，C++中一些`nullptr`的使用在 Rust 中根本不会出现。例如，移动对象不会留下需要销毁的任何东西。因此，没有必要使用`nullptr`作为可以调用`delete`或`free`的移动指针的占位符。

其他用法被`Option`所取代，在安全的 Rust 中，在访问包含的值之前需要检查空值情况。这种用法足够常见，以至于 Rust 在`Option`与引用（`&`或`&mut ref`）、`Box`（相当于`unique_ptr`）和`NonNull`（一个非空原始指针）一起使用时进行了优化。[Rust 文档](https://doc.rust-lang.org/std/option/index.html#representation)。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Null (nullptr))
