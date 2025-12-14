# 封装

> 原文：[`cel.cs.brown.edu/crp/idioms/encapsulation.html`](https://cel.cs.brown.edu/crp/idioms/encapsulation.html)

在 C++ 中，封装边界是类。在 Rust 中，封装边界是模块，它可能包含多个类型以及独立的函数。在更大的项目中，crate 也可能作为封装边界。

这种差异意味着在 Rust 中，更有可能存在多个紧密耦合的类型，这些类型在同一个模块中定义，并作为一个整体封装起来。

本节提供了在机械和概念上翻译 C++ 和 Rust 封装概念的方法。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Encapsulation)
