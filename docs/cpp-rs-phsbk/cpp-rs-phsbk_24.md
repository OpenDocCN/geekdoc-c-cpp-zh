# 私有构造函数

> [`cel.cs.brown.edu/crp/idioms/encapsulation/private_constructors.html`](https://cel.cs.brown.edu/crp/idioms/encapsulation/private_constructors.html)

在 C++ 中，可以通过声明为私有或使用 `class` 定义类并使用默认的私有可见性来使类的构造函数私有。

在 Rust 中，结构体的构造函数（实际的构造函数，而不是 "构造方法"）在类型及其所有字段可见的地方都是可见的。为了实现与 C++ 示例中类似的可见性限制，需要在 Rust 中的结构体中添加一个额外的私有字段。因为 Rust 支持零大小类型，所以额外的字段不会产生性能成本。[单元类型](https://doc.rust-lang.org/std/primitive.unit.html) 零大小，可以用于此目的。

```rs
#include <string>

struct Person {
  std::string name;
  int age;

private:
  Person() = default;
};

int main() {
  // fails to compile, Person::Person() private
  // Person nobody;

  // fails to compile since C++20
  // Person alice{"Alice", 42};
  return 0;
} 
```

```rs

mod person {

    pub struct Person {

        pub name: String,

        pub age: i32,

        _private: (),

    }

    impl Person {

        pub fn new(

            name: String,

            age: i32,

        ) -> Person {

            Person {

                name,

                age,

                _private: (),

            }

        }

    }

}

use person::*;

fn main() {

    // 结构 `person::Person` 的 `_private` 字段

    // 是私有的

    // let alice = Person {

    //     name: "Alice".to_string(),

    //     age: 42,

    //     _private: (),

    // };

    // 不能使用

    // 由于私有字段，使用结构字面量语法

    }

    //     name: "Bob".to_string(),

    //     age: 55,

    // };

    let carol =

        Person::new("Carol".to_string(), 20);

    // 可以匹配公共字段，然后

    // 使用 .. 忽略剩余的。

    let Person { name, age, .. } = carol;

}

```

## 枚举

与 C++ 联合不同，但与 `std::variant` 相似，Rust 枚举没有直接控制其变体或变体字段的可见性。在下面的示例中，`Shape` 联合的 `circle` 变体不是公开的，因此只能从 `Shape` 的定义内部访问，就像 `make_circle` 静态方法那样。

```rs
#include <iostream>

struct Triangle {
  double base;
  double height;
};

struct Circle {
  double radius;
};

union Shape {
  Triangle triangle;

private:
  Circle circle;

public:
  static Shape make_circle(double radius) {
    Shape s;
    s.circle = Circle(radius);
    return s;
  };
};

int main() {
  Shape triangle;
  triangle.triangle = Triangle{1.0, 2.0};
  Shape circle = Shape::make_circle(1.0);

  // fails to compile
  // circle.circle = Circle{1.0};

  // fails to compile
  // std::cout << shape.circle.radius;
} 
```

在 Rust 中，可见性修饰符不能应用于单个枚举变体或其字段。

```rs

mod shape {

    pub enum Shape {

        Triangle { base: f64, height: f64 },

        Circle { radius: f64 },

    }

}

use shape::*;

fn main() {

    // 尽管没有标记为 pub，变体构造函数仍然是可访问的。

    let triangle = Shape::Triangle {

        base: 1.0,

        height: 2.0,

    };

    let circle = Shape::Circle { radius: 1.0 };

    // 尽管没有标记为 pub，字段仍然是可访问的。

    match circle {

        Shape::Triangle { base, height } => {

            println!("Triangle: {}, {}", base, height);

        }

        Shape::Circle { radius } => {

            println!("Circle {}", radius);

        }

    }

}

```

相反，为了控制枚举实现的构建和模式匹配，可以采取两种方法之一。第一种控制字段的构建和访问，但不能检查哪个变体是活动的。

```rs

mod shape {

    pub struct Triangle {

        pub base: f64,

        pub height: f64,

        _private: (),

    }

    pub struct Circle {

        pub radius: f64,

        _private: (),

    }

    pub enum Shape {

        Triangle(Triangle),

        Circle(Circle),

    }

    impl Shape {

        pub fn new_triangle(base: f64, height: f64) -> Shape {

            Shape::Triangle(Triangle {

                base,

                height,

                _private: (),

            })

        }

        pub fn new_circle(radius: f64) -> Shape {

            Shape::Circle(Circle {

                radius,

                _private: (),

            })

        }

    }

}

use shape::*;

fn main() {

    let triangle = Shape::new_triangle(1.0, 2.0);

    let circle = Shape::new_circle(1.0);

    match circle {

        Shape::Triangle(Triangle { base, height, .. }) => {

            println!("三角形: {}, {}", base, height);

        }

        Shape::Circle(Circle { radius, .. }) => {

            println!("圆: {}", radius);

        }

    }

}

```

第二种方法将枚举放在具有私有字段的 struct 中，防止从模块外部进行构造和检查。

```rs

mod shape {

    enum ShapeKind {

        Triangle { base: f64, height: f64 },

        Circle { radius: f64 },

    }

    pub struct Shape(ShapeKind);

    impl Shape {

        pub fn new_circle(radius: f64) -> Shape {

            Shape(ShapeKind::Circle { radius })

        }

        pub fn new_triangle(base: f64, height: f64) -> Shape {

            Shape(ShapeKind::Triangle { base, height })

        }

        pub fn print(&self) {

            match self.0 {

                ShapeKind::Triangle { base, height } => {

                    println!("三角形: {}, {}", base, height);

                }

                ShapeKind::Circle { radius } => {

                    println!("圆: {}", radius);

                }

            }

        }

    }

}

use shape::*;

fn main() {

    let triangle = Shape::new_triangle(1.0, 2.0);

    let circle = Shape::new_circle(1.0);

    // 由于 Shape 具有私有字段，无法编译。

    // match circle {

    //   Shape(_) -> {}

    // }

    circle.print();

}

```

如果将变体设置为私有的目的是确保满足不变量，那么在保证封装结构体（Shape）使用时才保证不变量的情况下，暴露实现枚举（ShapeKind）但不暴露封装结构体的字段是有用的。在这种情况下，需要将字段设置为私有并定义一个获取器函数，因为否则字段将是可修改的，可能会违反封装结构体表示的不变量。

```rs

mod shape {

    pub enum ShapeKind {

        Triangle { base: f64, height: f64 },

        Circle { radius: f64 },

    }

    // Shape 的字段是私有的。

    pub struct Shape(ShapeKind);

    impl Shape {

        pub fn new(kind: ShapeKind) -> Option<Shape> {

            // ...检查不变量...

            Some(Shape(kind))

        }

        pub fn get_kind(&self) -> &ShapeKind {

            &self.0

        }

    }

}

use shape::*;

fn main() {

    let triangle = Shape::new(ShapeKind::Triangle {

        base: 1.0,

        height: 2.0,

    });

    let Some(circle) = Shape::new(ShapeKind::Circle { radius: 1.0 }) else {

        return;

    };

    // 由于 Shape 具有私有字段，无法编译。

    // match circle {

    //   Shape(c) => {}

    // };

    match circle.get_kind() {

        ShapeKind::Triangle { base, height } => {

            println!("三角形: {}, {}", base, height);

        }

        ShapeKind::Circle { radius } => {

            println!("圆: {}", radius);

        }

    }

}

```

Rust 中的情况类似于使用 C++的`std::variant`时的情形，对于`std::variant`，无法使变体本身私有。相反，可以使得构成变体的类型的构造函数私有，或者将变体封装在具有适当可见性控制的类中。

## [Rust 的`#[非穷尽性]`注解](#rusts-non_exhaustive-annotation)

如果一个结构体或枚举打算在[crate](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html)内部公开，但不应该在 crate 外部构造，那么可以使用`#[非穷尽性]`属性来约束构造。该属性可以应用于结构体和单个枚举变体，效果与添加私有字段相同。

然而，该属性在 crate 级别应用约束，而不是在模块级别。

```rs

#![allow(unused)] fn main() { #[非穷尽性]

pub struct Person {

    pub name: String,

    pub age: i32,

}

pub enum Shape {

    #[非穷尽性]

    Triangle { base: f64, height: f64 },

    #[非穷尽性]

    Circle { radius: f64 },

}

}

```

该属性通常用于强制库的客户端在匹配结构体字段时包含通配符，使得向结构体添加额外字段不会引起破坏性变更（即，在使用语义版本控制时不需要增加主版本组件[链接](https://doc.rust-lang.org/cargo/reference/semver.html)）。

将`#[非穷尽性]`属性应用于枚举本身，使得其中一个变体仿佛是私有的，在匹配变体本身时需要使用通配符。这在版本控制方面与应用于结构体时的效果相同，但不太有利。在大多数情况下，当添加新的枚举变体时，代码无法编译是期望的，因为这表明需要处理的新情况。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Private constructors)
