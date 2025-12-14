# 异常和错误处理

> 原文：[`cel.cs.brown.edu/crp/idioms/exceptions.html`](https://cel.cs.brown.edu/crp/idioms/exceptions.html)

在 C++中，调用者需要处理的错误有时通过哨兵值（例如，`std::map::find`生成一个空的迭代器）指示，有时通过异常（例如，`std::vector::at`抛出`std::out_of_range`），有时通过设置错误位（例如，`std::fstream::fail`）指示。不打算由调用者处理的错误通常通过异常指示（例如，`std::bad_cast`）。由于编程错误导致的错误通常只会导致未定义行为（例如，当索引超出范围时，`std::vector::operator[]`）。

相反，安全 Rust 有两个机制来指示错误。当错误预期由调用者处理（例如，由于用户输入）时，函数返回一个`Result`或`Option`。当错误是由于编程错误时，函数会引发恐慌。只有在使用不安全的 Rust 和未检查的函数变体时，才会发生未定义行为。

Rust 中的一些库将提供 API 的两个版本，一个返回`Result`或`Option`类型，另一个会引发恐慌，这样调用者可以选择错误的解释（预期的异常情况或程序员错误）。

使用`Result`或`Option`与使用异常之间的主要区别在于：

1.  `Result`和`Option`强制显式处理错误情况以访问包含的值。这也与 C++23 中的`std::expected`不同。

1.  在使用`Result`传播错误时，错误类型必须匹配。有一些库可以简化这一处理。

## `Result`与`Option`

本章中展示的 Rust 示例中的方法适用于`Result`和`Option`。当类型为`Option`时，表示在错误情况下没有额外的信息提供：`Option::None`不包含值，但`Result::Err`包含。当没有额外信息时，通常是因为只有一个可能导致错误情况的情况。

两种类型之间可以进行转换。

```rs

```

fn main() {

    let r: Result<i32, &'static str> =

        None.ok_or("my error message");

    let r2: Result<i32, &'static str> =

        None.ok_or_else(|| "expensive error message");

    let o: Option<i32> = r.ok();

}

```rs

```

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Exceptions and error handling)
