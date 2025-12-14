# 指向实现 (PIMPL)

> 原文：[`cel.cs.brown.edu/crp/patterns/pimpl.html`](https://cel.cs.brown.edu/crp/patterns/pimpl.html)

C++ 中的 [PIMPL 模式](https://en.cppreference.com/w/cpp/language/pimpl.html)通常用于通过从翻译单元的 ABI 中移除实现细节来提高编译时间。它还可以用于隐藏那些否则会在头文件中暴露的实现细节。

在 Rust 中，独立的编译单位是 crate，而不是文件或模块。在 crate 内部，编译器通过增量编译来最小化编译时间，而不是通过独立的编译。在 crate 之间，没有 Rust 原生 ABI 稳定性的保证，因此如果上游 crate 发生变化，下游 crate 需要重新编译。因此，出于性能考虑，PIMPL 模式不适用。

对于隐藏实现细节，而不是从头文件中排除细节，可以使用模块来控制可见性。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Pointer-to-implementation (PIMPL))
