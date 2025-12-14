# 设置器和获取器方法

> 原文：[`cel.cs.brown.edu/crp/idioms/encapsulation/setters_and_getters.html`](https://cel.cs.brown.edu/crp/idioms/encapsulation/setters_and_getters.html)

在 C++ 和 Rust 中，设置器和获取器的工作方式相似，但在 Rust 中使用频率较低。

在 C++ 中，看到以下二维向量的表示并不罕见，它隐藏了其实施细节，并提供了设置器和获取器来访问字段。这种选择通常是在需要稍后进行表示更改（例如，使用极坐标而不是直角坐标）时做出的，而不破坏客户端。

另一方面，在 Rust 中，此类类型几乎总是使用公共字段定义。

```rs
class Vec2 {
  double x;
  double y;

public:
  Vec2(double x, double y) : x(x), y(y) {}
  double getX() { return x; }
  double getY() { return y; }

  // ... vector operations ...
}; 
```

```rs

#![allow(unused)] fn main() { pub struct Vec2 {

    // 使用公共字段而不是获取器

    pub x: f64,

    pub y: f64,

}

impl Vec2 {

    // ... 向量操作 ...

}

}

```

差异的一个主要原因是借用检查器的限制。使用获取器函数时，整个结构体都被借用，从而阻止了对结构体其他字段的可变使用。

以下程序将无法编译，因为 `get_name()` 从 `alice` 中借用了所有内容。

```rs
struct Person {
    name: String,
    age: u32,
}

impl Person {
    fn get_name(&self) -> &String {
        &self.name
    }
}

fn main() {
    let mut alice = Person { name: "Alice".to_string(), age: 42 };
    let name = alice.get_name();

    alice.age = 43;

    println!("{}", name);
}
```

```rs
error[E0506]: cannot assign to `alice.age` because it is borrowed
  --> example.rs:16:5
   |
14 |     let name = alice.get_name();
   |                ----- `alice.age` is borrowed here
15 |
16 |     alice.age = 43;
   |     ^^^^^^^^^^^^^^ `alice.age` is assigned to here but it was already borrowed
17 |
18 |     println!("{}", name);
   |                    ---- borrow later used here

error: aborting due to 1 previous error 
```

一些导致方法差异的其他原因包括：

+   人体工程学：公共成员使得使用模式匹配成为可能。

+   性能透明度：表示的变化将显著改变获取器涉及的成本。暴露表示使成本变化可见。

+   对可变性的控制：静态生命周期检查可变引用消除了通过 Rust 的观察指针等效物对值进行意外修改的担忧。

## 具有不变性和新类型的类型

当类型需要保持不变性但希望暴露字段的好处时，可以使用新类型模式。定义一个包装的“新类型”结构体来表示具有不变性的数据，并通过非 `mut` 引用提供对底层结构体字段访问。

```rs

#![allow(unused)] fn main() { pub struct Vec2 {

    pub x: f64,

    pub y: f64,

}

/// 表示一个模量为 1 的 2-向量。

pub struct Normalized(Vec2); // 注意私有字段

fn sqrt_approx_zero(x: f64) -> bool {

    x < 0.001

}

impl Normalized {

    pub fn from_vec2(v: Vec2) -> Option<Self> {

        if sqrt_approx_zero(v.x * v.x + v.y * v.x - 1.0) {

            Some(Self(v))

        } else {

            None

        }

    }

    // 获取器提供了一个对底层 Vec2 值的引用

    // 不允许修改。

    pub fn get(&self) -> &Vec2 {

        &self.0

    }

}

}

```

## 从索引结构借用

由于获取器方法与借用检查器交互的方式，产生的一个显著限制是，无法使用 `Vec::get_mut` 等方法从像向量这样的索引结构中可变借用多个元素。

内置的索引类型有几种创建结构分割视图的方法。这些方法可以用来创建满足特定应用程序要求的辅助函数。

Rustonomicon 有 [实现此模式的示例](https://doc.rust-lang.org/nomicon/borrow-splitting.html)，使用了安全和不可安全 Rust。

## 设置方法

设置方法也会获取整个值，这会导致与返回可变引用的获取器相同的问题。与获取器方法一样，设置方法主要用于需要保持不变性时。

`<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Setter and getter methods)`
