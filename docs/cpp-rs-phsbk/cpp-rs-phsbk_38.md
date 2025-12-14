# 多重返回值

> [`cel.cs.brown.edu/crp/idioms/out_params/multiple_return.html`](https://cel.cs.brown.edu/crp/idioms/out_params/multiple_return.html)

C++中从函数或方法返回多个值的一个习语是传递引用，值可以赋给这些引用。

使用这种习语的原因有几个：

+   与 C++11 之前版本的兼容性，

+   在使用 C++风格的 C++代码库中工作，或者

+   性能问题。

将此程序翻译成 Rust 的惯用风格，使用了[元组](https://doc.rust-lang.org/std/primitive.tuple.html)或命名结构作为返回类型。

```rs
void get_point(int &x, int &y) {
  x = 5;
  y = 6;
}

int main() {
  int x, y;
  get_point(x, y);
  // ...
} 
```

```rs

fn get_point() -> (i32, i32) {

    (5, 6)

}

fn main() {

    let (x, y) = get_point();

    // ...

}

```

Rust 有一个专门的元组语法，并支持使用`let`绑定进行模式匹配，部分是为了支持此类用例。

## 直接翻译的问题

将使用输出参数的原例翻译成 Rust 是可能的，但 Rust 要求在将变量传递给函数之前初始化它们。生成的程序不是 Rust 的惯用风格。

```rs

// NOT IDIOMATIC RUST

fn get_point(x: &mut i32, y: &mut i32) {

    *x = 5;

    *y = 6;

}

fn main() {

    let mut x = 0; // 初始化为任意值

    let mut y = 0;

    get_point(&mut x, &mut y);

    // ...

}

```

这种方法需要为变量分配任意初始值，并使变量可变，这两者都使得编译器更难帮助避免编程错误。

此外，Rust 编译器针对优化程序的惯用版本进行了调整，并为该版本生成显著更快的二进制文件。

在内存分配性能是关注点的情况下（例如，当需要内存中重用整个缓冲区时），权衡可能不同。这种情况在关于预分配缓冲区的章节中进行了讨论。

## 与 C++11 及以后惯用 C++的相似性

在 C++11 及以后的版本中，`std::pair`和`std::tuple`可用于返回多个值，而不是将值赋给引用参数。

```rs
#include <tuple>
#include <utility>

std::pair<int, int> get_point() {
  return std::make_pair(5, 6);
}

int main() {
  int x, y;
  std::tie(x, y) = get_point();
  // ...
} 
```

这更符合 Rust 返回多个值的正常惯用风格。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Multiple return values)
