# 重载

> 原文：[`cel.cs.brown.edu/crp/idioms/overloading.html`](https://cel.cs.brown.edu/crp/idioms/overloading.html)

C++支持函数重载，只要函数调用可以通过其参数的数量或类型来区分。

Rust 不支持这种类型的函数重载。相反，Rust 有一些不同的机制（其中一些 C++也有）来实现重载的效果，并且与类型推断更好地交互。这些机制通常涉及在代码中使重载函数之间的共同点明显。

```rs
#include <string>

double twice(double x) {
  return x + x;
}

int twice(int x) {
  return x + x;
} 
```

```rs

```

#![allow(unused)] fn main() { fn twice(x: f64) -> f64 {

    x + x

}

// error[E0428]: the name `twice` is defined multiple times

// fn twice(x: i32) -> i32 {

//     x + x

// }

}

```rs

```

实际上，即使是在 C++中，上述示例也可能会以更结构化的方式实现，甚至使用模板。

这样表述后，示例可以翻译成 Rust，值得注意的是，需要为类型添加特性约束。

```rs
template <typename T>
T twice(T x) {
  return x + x;
} 
```

```rs

```

#![allow(unused)] fn main() { fn twice<T>(x: T) -> T::Output

where

    T: std::ops::Add<T>,

    T: Copy,

{

    x + x

}

}

```rs

```

## 重载方法

在 C++中，可以在同一类型上具有具有相同名称但不同签名的多个方法。在 Rust 中，每个特性实现最多只能有一个具有相同名称的方法，对于类型来说，最多只能有一个具有相同名称的固有方法。

当存在多个具有相同名称的方法，因为方法为多个特性定义时，必须在调用位置通过指定特性来区分所需的方法。

```rs

```

trait TraitA {

    fn go(&self) -> String;

}

trait TraitB {

    fn go(&self) -> String;

}

struct MyStruct;

impl MyStruct {

    fn go(&self) -> String {

        "称为固有方法".to_string()

    }

}

impl TraitA for MyStruct {

    fn go(&self) -> String {

        "称为特性 A 方法".to_string()

    }

}

impl TraitB for MyStruct {

    fn go(&self) -> String {

        "称为特性 B 方法".to_string()

    }

}

fn main() {

    let my_struct = MyStruct;

    // 调用固有方法

    println!("{}", my_struct.go());

    // 调用来自 TraitA 的方法

    println!("{}", TraitA::go(&my_struct));

    // 调用来自 TraitB 的方法

    println!("{}", TraitB::go(&my_struct));

}

```rs

```

这种情况的一个例外是，当方法都来自同一个泛型特性，并且实现有不同的类型参数时。在这种情况下，如果签名足以确定要使用哪个实现，则不需要指定特性来解析方法。这在使用[`From`特性](https://doc.rust-lang.org/std/convert/trait.From.html)时很常见。

```rs

```

struct Widget;

impl From<i32> for Widget {

    fn from(x: i32) -> Widget {

        Widget

    }

}

impl From<f32> for Widget {

    fn from(x: f32) -> Widget {

        Widget

    }

}

fn main() {

    // 调用<Widget as From<i32>>::from

    let w1 = Widget::from(5);

    // 调用<Widget as From<f32>>::from

    let w2 = Widget::from(1.0);

}

```rs

```

## 重载运算符

在 C++ 中，大多数运算符可以通过自由函数或通过在类上定义运算符的方法来重载。

Rust 通过特定特质的实现提供运算符。如果特质没有实现，即使实现了与特质要求同名的方法，也无法使类型与运算符一起使用。

```rs
struct Vec2 {
  double x;
  double y;

  Vec2 operator+(const Vec2 &other) const {
    return Vec2{x + other.x, y + other.y};
  }
};

int main() {
  Vec2 a{1.0, 2.0};
  Vec2 b{3.0, 4.0};
  Vec2 c = a + b;
} 
```

```rs

```

#[derive(Clone, Copy)]

struct Vec2 {

    x: f64,

    y: f64,

}

impl std::ops::Add for &Vec2 {

    type Output = Vec2;

    // 注意，这里的 self 类型是 &Vec2。

    fn add(self, other: Self) -> Vec2 {

        Vec2 {

            x: self.x + other.x,

            y: self.y + other.y,

        }

    }

}

fn main() {

    let a = Vec2 { x: 1.0, y: 2.0 };

    let b = Vec2 { x: 3.0, y: 4.0 };

    let c = &a + &b;

}

```rs

```

此外，有时最好为各种引用类型的组合提供特质实现，特别是对于实现 `Copy 特质` 的类型，因为它们可能希望使用或不需要引用。对于上面的例子，需要定义四个实现。

```rs

```

#[derive(Clone, Copy)]

struct Vec2 {

    x: f64,

    y: f64,

}

impl std::ops::Add<&Vec2> for &Vec2 {

    type Output = Vec2;

    fn add(self, other: &Vec2) -> Vec2 {

        Vec2 {

            x: self.x + other.x,

            y: self.y + other.y,

        }

    }

}

// 如果 Vec2 不那么小，可能希望在下面重用空间

// 实现细节，因为它们需要所有者。

impl std::ops::Add<Vec2> for &Vec2 {

    type Output = Vec2;

    fn add(self, other: Vec2) -> Vec2 {

        Vec2 {

            x: self.x + other.x,

            y: self.y + other.y,

        }

    }

}

impl std::ops::Add<&Vec2> for Vec2 {

    type Output = Vec2;

    fn add(self, other: &Vec2) -> Vec2 {

        Vec2 {

            x: self.x + other.x,

            y: self.y + other.y,

        }

    }

}

impl std::ops::Add<Vec2> for Vec2 {

    type Output = Vec2;

    fn add(self, other: Vec2) -> Vec2 {

        Vec2 {

            x: self.x + other.x,

            y: self.y + other.y,

        }

    }

}

fn main() {

    let a = Vec2 { x: 1.0, y: 2.0 };

    let b = Vec2 { x: 3.0, y: 4.0 };

    let c = a + b;

}

```rs

```

通过定义宏可以解决重复问题。

```rs

```

#[derive(Clone, Copy)]

struct Vec2 {

    x: f64,

    y: f64,

}

macro_rules! impl_add_vec2 {

    ($lhs:ty, $rhs:ty) => {

        impl std::ops::Add<$rhs> for $lhs {

            type Output = Vec2;

            fn add(self, other: $rhs) -> Vec2 {

                Vec2 {

                    x: self.x + other.x,

                    y: self.y + other.y,

                }

            }

        }

    };

}

impl_add_vec2!(&Vec2, &Vec2);

impl_add_vec2!(&Vec2, Vec2);

impl_add_vec2!(Vec2, &Vec2);

impl_add_vec2!(Vec2, Vec2);

fn main() {

    let a = Vec2 { x: 1.0, y: 2.0 };

    let b = Vec2 { x: 3.0, y: 4.0 };

    let c = a + b;

}

```rs

```

## 默认参数

C++ 中的默认参数有时是通过函数重载来实现的。

Rust 没有默认参数。相反，可以使用 `Option` 类型的参数来提供类似的效果。

```rs
unsigned int shift(unsigned int x,
                   unsigned int shiftAmount) {
  return x << shiftAmount;
}

unsigned int shift(unsigned int x) {
  return shift(x, 2);
}

int main() {
  unsigned int a = shift(7); // shifts by 2
} 
```

```rs

```

use std::ops::Shl;

fn shift(

    x: u32,

    shift_amount: Option<u32>,

) -> u32 {

    let a = shift_amount.unwrap_or(2);

    x.shl(a)

}

fn main() {

    let res = shift(7, None); // 向右移位 2

}

```rs

```

## 无关的重载

Rust 中完全的临时重载缺乏鼓励了定义特质，这些特质捕捉了类型之间的基本共同性，因此可以在这些接口上实现函数，并广泛使用。然而，这也有时鼓励了反模式，即定义仅捕捉偶然共同性的特质（例如具有相同名称的方法）。

在这些情况下，简单地定义单独的函数是更好的编程实践，而不是强行塞入一个没有真正共同性的特质。

这在 Rust 中常见于构造器静态方法的命名约定。它们不是都命名为 `new` 并带有不同的参数，而是通常被赋予形式为 `from_something` 的名称（[通常是这样的命名方式](https://rust-lang.github.io/api-guidelines/naming.html)），其中 `something` 根据构造的值而变化，或者如果适当，则使用更具体的名称。

```rs

```

#![allow(unused)] fn main() { struct Vec3 {

    x: f64,

    y: f64,

    z: f64,

}

impl Vec3 {

    fn from_x(x: f64) -> Vec3 {

        Vec3 { x, y: 0.0, z: 0.0 }

    }

    fn from_y(y: f64) -> Vec3 {

        Vec3 { x: 0.0, y, z: 0.0 }

    }

    fn diagonal(d: f64) -> Vec3 {

        Vec3 { x: d, y: d, z: d }

    }

}

}

```rs

```

这与 `From` 和 `Into` 特质支持的转换方法不同，这些方法具有额外的目的，即支持泛型函数上的特质界限，这些函数应该接受任何可转换为特定类型的类型。

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Overloading)
