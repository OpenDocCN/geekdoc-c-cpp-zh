# 适配器模式

> 原文：[`cel.cs.brown.edu/crp/patterns/adapter.html`](https://cel.cs.brown.edu/crp/patterns/adapter.html)

在 C++ 中，如果现有的类需要实现一个新的接口，通常使用适配器模式。该模式涉及定义一个包装类，通过委派到原始类的方法来实现接口。

在 Rust 中，相同的模式是可能的，并且有时由于孤儿规则（[`doc.rust-lang.org/book/ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types`](https://doc.rust-lang.org/book/ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types)）的需要而变得必要。然而，因为特质可以在类型或特质定义的地方实现，通常可以直接为类型实现特质，而不需要包装器。

以下示例向 C++ 中的 `std::string` 和 Rust 中的 `String` 添加了一个接口，以便与库中定义的模板函数一起使用。当需要动态分派时，Rust 中实现现有类型的特质的实现方式没有变化。

```rs
#include <concepts>
#include <iostream>
#include <string>

template <typename T>
concept doubleable = requires(const T t) {
  { t.twice() } -> std::same_as<T>;
};

template <doubleable T>
T quadruple(const T &x) {
  return x.twice().twice();
}

struct DoubleableString {
  std::string str;

  DoubleableString twice() const {
    return DoubleableString{this->str +
                            this->str};
  }
};

int main() {
  auto s = quadruple(
      DoubleableString{std::string("a")});
  std::cout << s.str << std::endl;
} 
```

```rs

trait Doubleable {

    fn twice(&self) -> Self;

}

impl Doubleable for String {

    fn twice(&self) -> Self {

        self.clone() + self

    }

}

fn quadruple<T: Doubleable>(x: T) -> T {

    x.twice().twice()

}

fn main() {

    let s = quadruple(String::from("a"));

    println!("{}", s);

}

```

## 扩展特性

在 C++ 中，向现有类型添加功能通常是通过定义额外的函数来实现的。

Rust 可以通过定义独立函数来添加功能。Rust 还支持向现有类型添加方法的能力。它是通过使用前面章节中描述的相同机制来实现的。通过使用泛型实现，甚至可以向实现了一些其他特质的任何类型添加方法。这是 `itertools` crate（[`docs.rs/itertools/latest/itertools/`](https://docs.rs/itertools/latest/itertools/)）添加对实现 `Iterator` 特质的任何类型额外功能所采用的方法。

```rs

trait Middle {

    type Output;

    fn middle(&mut self) -> Option<Self::Output>;

}

impl<T: ExactSizeIterator> Middle for T {

    type Output = T::Item;

    fn middle(&mut self) -> Option<Self::Output> {

        let len = self.len();

        if len > 0 && len % 2 == 1 {

            self.nth(len / 2)

        } else {

            None

        }

    }

}

fn main() {

    println!("{:?}", [1, 2, 3].iter().middle());

    println!("{:?}", [1, 2, 3].iter().map(|n| n + 1).middle());

}

```

`map` 方法返回的类型与 `iter` 不同，但可以在两者的结果上调用 `middle`。

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Adapter pattern)
