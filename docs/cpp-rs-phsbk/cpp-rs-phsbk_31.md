# 用户定义的转换

> 原文：[`cel.cs.brown.edu/crp/idioms/user-defined_conversions.html`](https://cel.cs.brown.edu/crp/idioms/user-defined_conversions.html)

在 C++ 中，用户定义的转换是通过 [转换构造函数](https://en.cppreference.com/w/cpp/language/converting_constructor) 或 [转换函数](https://en.cppreference.com/w/cpp/language/cast_operator) 创建的。因为转换构造函数是可选的（通过 `explicit` 指示符），在 C++ 代码中隐式转换经常发生。在以下示例中，赋值和函数调用都使用了由转换构造函数提供的隐式转换。

Rust 显著减少了隐式转换的使用。相反，大多数转换都是显式的。`std::convert` 模块提供了几个 trait 用于处理用户定义的转换。在 Rust 中，以下示例通过实现 `From` trait 使用显式转换。

```rs
struct Widget {
  Widget(int) {}
  Widget(int, int) {}
};

void process(Widget w) {}

int main() {
  Widget w1 = 1;
  Widget w2 = {4, 5};
  process(1);
  process({4, 5});

  return 0;
} 
```

```rs

struct Widget;

impl From<i32> for Widget {

    fn from(_x: i32) -> Widget {

        Widget

    }

}

impl From<(i32, i32)> for Widget {

    fn from(_x: (i32, i32)) -> Widget {

        Widget

    }

}

fn process(w: Widget) {}

fn main() {

    let w1: Widget = 1.into();

    // 对于构造来说，这是更常见的做法：

    let w1b = Widget::from(1);

    let w2: Widget = (4, 5).into();

    // 对于构造来说，这是更常见的做法：

    let w2b = Widget::from((4, 5));

    process(1.into());

    process((4, 5).into());

}

```

上面的 `into` 方法是通过为实现了 `From` trait 的类型提供的 [泛型实现](https://doc.rust-lang.org/book/ch10-02-traits.html#using-trait-bounds-to-conditionally-implement-methods) 来提供的。由于存在 [泛型实现](https://doc.rust-lang.org/std/convert/trait.Into.html#impl-Into%3CU%3E-for-T)，通常更倾向于实现 `From` trait 而不是 `Into` trait，并让 `Into` trait 由该泛型实现提供。

## 转换函数

C++ 的转换函数允许从定义的类到其他类型的转换。

要在 Rust 中实现相同的功能，可以在相反方向上实现 `From` trait。至少源类型或目标类型中的一种必须在与 trait 实现相同的 crate 中定义。

```rs
#include <utility>

struct Point {
  int x;
  int y;

  operator std::pair<int, int>() const {
    return std::pair(x, y);
  }
};

void process(std::pair<int, int>) {}

int main() {
  Point p1{1, 2};
  Point p2{3, 4};

  std::pair<int, int> xy = p1;
  process(p2);

  return 0;
} 
```

```rs

struct Point {

    x: i32,

    y: i32,

}

impl From<Point> for (i32, i32) {

    fn from(p: Point) -> (i32, i32) {

        (p.x, p.y)

    }

}

fn process(x: (i32, i32)) {}

fn main() {

    let p1 = Point { x: 1, y: 2 };

    let p2 = Point { x: 3, y: 4 };

    let xy: (i32, i32) = p1.into();

    process(p2.into());

}

```

转换函数通常用于在 C++ 中实现安全的布尔模式，而在 Rust 中以不同的方式处理。

## 借用转换

`From` 和 `Into` 特性中的方法会获取要转换的值的所有权。当在 C++ 中不希望这样做时，转换函数可以只接受并返回引用。

为了在 Rust 中实现相同的功能，使用 `AsRef` 特性[`doc.rust-lang.org/std/convert/trait.AsRef.html`](https://doc.rust-lang.org/std/convert/trait.AsRef.html) 或 `AsMut` 特性[`doc.rust-lang.org/std/convert/trait.AsMut.html`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)。

```rs
#include <iostream>
#include <string>

struct Person {
  std::string name;

  operator std::string &() {
    return this->name;
  }
};

void process(const std::string &name) {
  std::cout << name << std::endl;
}

int main() {
  Person alice{"Alice"};

  process(alice);

  return 0;
} 
```

```rs

struct Person {

    name: String,

}

impl AsRef<str> for Person {

    fn as_ref(&self) -> &str {

        &self.name

    }

}

fn process(name: &str) {

    println!("{}", name);

}

fn main() {

    let alice = Person {

        name: "Alice".to_string(),

    };

    process(alice.as_ref());

}

```

在函数定义中，通常使用 `AsRef` 或 `AsMut` 作为特性界限。使用带有 `AsRef` 或 `AsMut` 界限的泛型允许客户端使用任何可以廉价地视为函数想要与之一起工作的类型的任何内容来调用函数。使用这种技术，上述 `process` 的定义可以像以下示例中那样定义。

```rs

struct Person {

name: String, }   impl AsRef<str> for Person {

fn as_ref(&self) -> &str { &self.name } }   fn process<T: AsRef<str>>(name: T) {

    println!("{}", name.as_ref());

}

fn main() {

    let alice = Person {

        name: "Alice".to_string(),

    };

    process(alice);

}

```

这种技术通常与接受文件系统路径的函数一起使用，这样就可以更轻松地将文本字符串用作路径。

## 可失败转换

在 C++ 中，当转换可能失败时，从转换构造函数或转换函数抛出异常是可能的（尽管通常不鼓励这样做）。

Rust 中的错误处理不使用异常。相反，使用 `TryFrom` 特性[`doc.rust-lang.org/std/convert/trait.TryFrom.html`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html) 和 `TryInto` 特性[`doc.rust-lang.org/std/convert/trait.TryInto.html`](https://doc.rust-lang.org/std/convert/trait.TryInto.html) 来进行可能失败的转换。这些特性与 `From` 和 `Into` 不同，因为它们返回一个 `Result`，这可能会指示一个失败的情况。当一个转换可能失败时，应该实现 `TryFrom` 并依赖客户端调用结果上的 `unwrap`，而不是在 `From` 实现中恐慌。

```rs
#include <stdexcept>
#include <string>

class NonEmpty {
  std::string s;

public:
  NonEmpty(std::string s) : s(s) {
    if (this->s.empty()) {
      throw std::domain_error("empty string");
    }
  }
};

int main() {
  std::string s("");
  NonEmpty x = s; // throws

  return 0;
} 
```

```rs

use std::convert::TryFrom;

use std::convert::TryInto;

struct NonEmpty {

    s: String,

}

#[derive(Clone, Copy, Debug)]

struct NonEmptyStringError;

impl TryFrom<String> for NonEmpty {

    type Error = NonEmptyStringError;

    fn try_from(

        s: String,

    ) -> Result<NonEmpty, NonEmptyStringError>

    {

        if s.is_empty() {

            Err(NonEmptyStringError)

        } else {

            Ok(NonEmpty { s })

        }

    }

}

fn main() {

    let res: Result<

        NonEmpty,

        NonEmptyStringError,

    > = "".to_string().try_into();

    match res {

        Ok(ne) => {

            println!("Converted!");

        }

        Err(err) => {

            println!("Couldn't convert");

        }

    }

}

```

就像 `From` 和 `Into` 一样，对于实现了 `TryFrom` 的所有内容，都有一个 `TryInto` 的泛型实现[`doc.rust-lang.org/std/convert/trait.TryInto.html#impl-TryInto%3CU%3E-for-T`](https://doc.rust-lang.org/std/convert/trait.TryInto.html#impl-TryInto%3CU%3E-for-T)。

## 隐式转换

Rust 确实有一种用户定义的隐式转换，称为 [deref coercions](https://doc.rust-lang.org/std/ops/trait.Deref.html#deref-coercion)，由 `Deref` 特性 ([`Deref` trait](https://doc.rust-lang.org/std/ops/trait.Deref.html)) 和 `DerefMut` 特性 ([`DerefMut` trait](https://doc.rust-lang.org/std/ops/trait.DerefMut.html)) 提供。这些转换是为了使指针类型更易于使用。

Rust 书籍中给出了实现自定义指针类型特性的 [示例](https://doc.rust-lang.org/book/ch15-02-deref.html)。

## 总结

在 `std::convert` 模块的文档中给出了何时使用哪种转换接口的总结 ([`std::convert` 模块](https://doc.rust-lang.org/std/convert/index.html))。

`<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=User-defined conversions)`
