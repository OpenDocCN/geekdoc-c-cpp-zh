# 三/五/零规则

> 原文：[`cel.cs.brown.edu/crp/idioms/constructors/rule_of_three_five_zero.html`](https://cel.cs.brown.edu/crp/idioms/constructors/rule_of_three_five_zero.html)

## 三条规则

在 C++ 中，三条规则是一个经验法则，如果一个类有一个用户定义的析构函数、复制构造函数或复制赋值运算符，它可能应该有所有这三个。

对于 Rust，相应的规则是，如果一个类型有用户定义的 `Clone` 或 `Drop` 实现，它可能需要两者。这是与 C++ 中的三条规则相同的原因：如果一个类型有用户定义的 `Clone` 或 `Drop` 实现，这通常是因为该类型管理资源，`Clone` 和 `Drop` 都需要为资源执行特殊操作。

## 五条规则

C++ 中的五条规则指出，如果一个类型需要用户定义的复制构造函数或复制赋值运算符的移动语义，那么也应该提供用户定义的移动构造函数和移动赋值运算符，因为没有隐式的移动构造函数或移动赋值运算符会被生成。

在 Rust 中，由于 C++ 和 Rust 之间移动语义的差异，这条规则不适用。差异在 C++ 和 Rust 之间的移动语义。

## 零规则

零规则指出，具有用户定义的复制/移动构造函数、赋值运算符和析构函数的类应仅处理所有权，其他类不应有这些构造函数或析构函数。在实践中，大多数类应使用 STL（`shared_ptr`、`vector` 等）中的类型来处理所有权问题，这样隐式定义的复制和移动构造函数就足够了。

在 Rust 中，情况也是如此。请参阅 Rust 类型等价列表，以了解 C++ 智能指针类型 和 C++ 容器类型 的等价类型。

在应用零规则时，C++ 和 Rust 之间有一个区别是，在 C++ 中 `std::unique_ptr` 可以接受自定义的删除器，这使得可以使用 `std::unique_ptr` 来包装需要自定义销毁逻辑的原始指针。在 Rust 中，`Box` 类型不是以相同的方式进行参数化的。为了达到相同的目标，必须定义一个新的类型，并实现用户定义的 `Drop`，正如在复制和移动构造函数章节中的示例中所做的那样。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Rule of three/five/zero)
