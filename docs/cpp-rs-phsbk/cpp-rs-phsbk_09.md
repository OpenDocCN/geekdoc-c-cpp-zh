# 抽象类、接口和动态分派

> 原文：[`cel.cs.brown.edu/crp/idioms/data_modeling/abstract_classes.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/abstract_classes.html)

在 C++ 中，当使用动态分派来解析调用的方法时，接口通过抽象类定义。实现该接口的类型从抽象类继承。在 Rust 中，接口由 *trait* 提供，然后为支持该 trait 的类型实现该 trait。程序可以编写在 *trait 对象* 上，这些对象使用该 trait 作为其基类型。

以下示例定义了一个接口、该接口的两个实现以及一个接受满足接口的参数的函数。在 C++ 中，接口通过具有纯虚方法的抽象类定义，而在 Rust 中，接口通过 trait 定义。在这两种语言中，函数（C++ 中的 `printArea` 和 Rust 中的 `print_area`）都使用动态分派调用方法。

```rs
#include <iostream>
#include <memory>

// Define an abstract class for an interface
struct Shape {
  Shape() = default;
  virtual ~Shape() = default;
  virtual double area() = 0;
};

// Implement the interface for a concrete class
struct Triangle : public Shape {
  double base;
  double height;

  Triangle(double base, double height)
      : base(base), height(height) {}

  double area() override {
    return 0.5 * base * height;
  }
};

// Implement the interface for a concrete class
struct Rectangle : public Shape {
  double width;
  double height;

  Rectangle(double width, double height)
      : width(width), height(height) {}

  double area() override {
    return width * height;
  }
};

// Use an object via a reference to the interface
void printArea(Shape &shape) {
  std::cout << shape.area() << std::endl;
}

int main() {
  Triangle triangle = Triangle{1.0, 1.0};

  printArea(triangle);

  // Use an object via an owned pointer to the
  // interface
  std::unique_ptr<Shape> shape;
  if (true) {
    shape = std::make_unique<Rectangle>(1.0, 1.0);
  } else {
    shape = std::make_unique<Triangle>(
        std::move(triangle));
  }

  // Convert to a reference to the interface
  printArea(*shape);
} 
```

```rs

// 定义一个接口

trait Shape {

    fn area(&self) -> f64;

}

struct Triangle {

    base: f64,

    height: f64,

}

// 为具体类型实现接口

impl Shape for Triangle {

    fn area(&self) -> f64 {

        0.5 * self.base * self.height

    }

}

struct Rectangle {

    width: f64,

    height: f64,

}

// 为具体类型实现接口

impl Shape for Rectangle {

    fn area(&self) -> f64 {

        self.width * self.height

    }

}

// 通过接口的引用使用一个值

fn print_area(shape: &dyn Shape) {

    println!("{}", shape.area());

}

fn main() {

    let triangle = Triangle {

        base: 1.0,

        height: 1.0,

    };

    print_area(&triangle);

    // 通过拥有指针使用一个值

    // 接口

    let shape: Box<dyn Shape> = if true {

        Box::new(Rectangle {

            width: 1.0,

            height: 1.0,

        })

    } else {

        Box::new(triangle)

    };

    // 转换为接口的引用

    }

}

```

Rust 的实现与 C++ 的实现略有不同的地方有几个。

在 Rust 中，只要 trait 本身可见，其方法总是可见的。此外，类型实现 trait 的这一事实总是可见的，只要 trait 和类型都可见。Rust 的这些特性解释了为什么在某些地方可能会找到可见性声明，但在 Rust 中却没有。

在 C++ 中，要关联方法与类型而不是该类型的值，您使用 `static` 关键字。在 Rust 中，非静态方法需要一个显式的 `self` 参数。这种语法选择使得可以指示（以与其他参数类似的方式）方法是否修改对象（通过使用 `&mut self` 而不是 `&self`）以及它是否拥有对象（通过使用 `self` 而不是 `&self`）。

Rust 方法不需要声明为虚拟。由于 vtable 表示的不同，一个类型的所有方法都可用于动态分发。使用 vtable 的值的类型用 `dyn` 关键字表示。这将在下面进一步描述。

此外，Rust 没有与虚拟析构函数声明等效的功能，因为在 Rust 中，每个 vtable 都包含用于值的析构行为（无论是否由用户定义的 `Drop` 实现提供）。

## Vtables 和 Rust 特性对象类型

C++ 和 Rust 都需要某种形式的间接引用来对接口执行动态分发。在 C++ 中，这种间接引用的形式是抽象类的指针（而不是派生具体类的指针），并使用 vtable 来解析虚拟方法。

在上述 Rust 示例中，类型 `dyn Shape` 是 `Shape` 特性的特性对象类型。特性对象包含 vtable 和底层值。

在 C++ 中，所有从具有虚拟方法的类继承的类的对象在其表示中都有一个 vtable，无论是否使用动态分发。对象的指针或引用的大小与没有虚拟方法的对象的指针大小相同，但每个对象都包含其 vtable。

在 Rust 中，vtable 仅在值表示为特性对象时存在。特性对象的引用是正常引用的两倍大小，因为它包含指向值的指针和指向 vtable 的指针。在上面的 Rust 示例中，`main` 中的局部变量 `triangle` 在其表示中没有 vtable，但当将其引用转换为特性对象引用（以便可以传递给 `print_area`）时，它确实包含指向 vtable 的指针。

此外，就像 C++ 中的抽象类不能用作局部变量的类型、函数参数的类型或函数返回值的类型一样，Rust 中的特性对象类型也不能在相应的上下文中使用。在 Rust 中，这是通过类型 `dyn Shape` 不实现 `Sized` 标记特性来强制执行的，防止它在需要静态地知道类型大小的上下文中使用。

以下示例展示了由于未实现 `Sized`，特性对象类型可以在某些地方使用，而在其他地方则不能。在 Rust 中被禁止使用的情况在 C++ 中也会被禁止，因为 `Shape` 是一个抽象类。

```rs

trait Shape {

fn area(&self) -> f64; }   struct Triangle {

base: f64, height: f64, }   impl Shape for Triangle {

fn area(&self) -> f64 { 0.5 * self.base * self.height } }   fn main() {

    // 局部变量必须具有已知的大小。

    // let v: dyn Shape = Triangle { base: 1.0, height: 1.0 };

    // 引用总是具有已知的大小。

    let shape: &dyn Shape = &Triangle {

        base: 1.0,

        height: 1.0,

    };

    // 盒子也总是具有已知的大小。

    let boxed_shape: Box<dyn Shape> = Box::new(Triangle {

        base: 1.0,

        height: 1.0,

    });

    // Types like Option<T> include the value of type T directly, and so also

    // need to know the size of T.

    // let v: Option<dyn Shape> = Some(Triangle { base: 1.0, height: 1.0 });

}

// Parameter types must have a known size.

// fn print_area(shape: dyn Shape) { }

fn print_area(shape: &dyn Shape) {}

```

The decision to include the vtable in the reference instead of in the value is one part of what makes it reasonable to use traits both for polymorphism via dynamic dispatch and for polymorphism via static dispatch, where one would use concepts in C++.

## Rust 中特质对象的局限性

In Rust, not all traits can be used as the base trait for trait objects. The most commonly encountered restriction is that traits that require knowledge of the object's size via a `Sized` supertrait are not `dyn`-compatible. There are [additional restrictions](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility).

## 特质对象和生命周期

Objects which are used with dynamic dispatch may contain pointers or references to other objects. In C++ the lifetimes of those references must be tracked manually by the programmer.

Rust checks the bounds on the lifetimes of references that the trait objects may contain. If the bounds are not given explicitly, they are determined according to the [lifetime elision rules](https://doc.rust-lang.org/reference/lifetime-elision.html#r-lifetime-elision.trait-object). The bound is part of the type of the trait object.

Usually the elision rules pick the correct lifetime bound. Sometimes, the rules result in surprising error messages from the compiler. In those situations or when the compiler cannot determine which lifetime bound to assign, the bound may be given manually. The following example shows explicitly what the inferred lifetimes are for a structure storing a trait object and for the `print_area` function.

```rs

trait Shape {

fn area(&self) -> f64; }   struct Triangle {

base: f64, height: f64, }   impl Shape for Triangle {

fn area(&self) -> f64 { 0.5 * self.base * self.height } }   struct Scaled {

    scale: f64,

    // 'static is the lifetime that would be inferred by the lifetime elision

    // rule [lifetime-elision.trait-object.default].

    shape: Box<dyn Shape + 'static>,

}

impl Shape for Scaled {

    fn area(&self) -> f64 {

        self.scale * self.shape.area()

    }

}

// These are the lifetimes that would be inferred by the lifetime elision rule

// [lifetime-elision.function.implicit-lifetime-parameters] for the reference

// and [lifetime-elision.trait-object.containing-type-unique] for the trait

// bound.

fn print_area<'a>(shape: &'a (dyn Shape + 'a)) {

    println!("{}", shape.area());

}

fn main() {

    let triangle = Triangle {

        base: 1.0,

        height: 1.0,

    };

    print_area(&triangle);

    let scaled_triangle = Scaled {

        scale: 2.0,

        shape: Box::new(triangle),

    };

    print_area(&scaled_triangle);

}

```

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=抽象类、接口和动态分派)
