# 概念、接口和静态分发

> [`cel.cs.brown.edu/crp/idioms/data_modeling/concepts.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/concepts.html)

在 C++中，通过实现一个模板函数或模板方法与类型交互，从而在接口上实现静态分发。

下面示例中的模板函数`twiceArea`使用了模板类型参数上的`area()`方法。

要在 Rust 中实现相同的目标，需要定义一个特性和所需的方法（`twice_area`），并将特性和方法用作泛型函数类型参数的约束。

```rs
#include <iostream>

struct Triangle {
  double base;
  double height;

  Triangle(double base, double height)
      : base(base), height(height) {}

  // NOT virtual: it will be used with static
  // dispatch
  double area() const {
    return 0.5 * base * height;
  }
};

// Generic function using interface
template <class T>
double twiceArea(const T &shape) {
  return shape.area() * 2;
}

int main() {
  Triangle triangle{1.0, 1.0};

  std::cout << twiceArea(triangle) << std::endl;
  return 0;
} 
```

```rs

// 泛型函数将使用的接口

trait Shape {

    fn area(&self) -> f64;

}

struct Triangle {

    base: f64,

    height: f64,

}

// 为类型实现接口

impl Shape for Triangle {

    fn area(&self) -> f64 {

        0.5 * self.base * self.height

    }

}

// 使用接口的泛型函数

fn twice_area<T: Shape>(shape: &T) -> f64 {

    2.0 * shape.area()

}

fn main() {

    let triangle = Triangle {

        base: 1.0,

        height: 1.0,

    };

    println!("{}", twice_area(&triangle));

}

```

注意，在 Rust 示例中，特性和结构的定义与第虚拟方法和动态分发章节中的示例没有变化。即便如此，这个例子确实使用了静态分发。这是 Rust 在 vtables 和 vptrs 表示方面的设计权衡的结果，该结果将在该章节稍后描述。

上面的示例中，Rust 与 C++之间的差异源于 Rust 的名义类型（类型必须选择支持特定的接口，仅仅有正确的方法是不够的）和 C++的模板元编程，它实现了一种结构化或鸭子类型（类型只需要有实际使用的方法，无需显式选择支持接口）。

## 模板与泛型函数

Rust 之所以采用名义类型而非结构类型，与 C++模板和 Rust 泛型函数之间的差异有关。特别是，C++模板只有在所有模板参数都提供并完全展开后才会进行类型检查，而 Rust 泛型函数则独立于类型参数进行类型检查。

由于函数在类型参数已知之前就已经进行检查，因此那些类型值可以应用的方法和函数也必须在类型参数已知之前就已知。

在编程语言设计空间中，这个点更倾向于推理这些函数的简单性，而不是模板编程方法带来的灵活性。当编写既提供基于其他泛型函数定义的泛型函数的库时，这一点尤其有价值，因为 C++编译器可以提供的静态保证要少得多，因为不可能测试所有可能的实例化。

然而，在 C++和 Rust 中，编译器都会生成多个实现以实现静态分派。

## C++约束和概念

Rust 对接口进行静态分派的方法可以通过对[C++概念](https://en.cppreference.com/w/cpp/language/constraints)的严格应用部分（但只是部分）地进行模拟。

应用概念的传统方式仍然是结构化的，并不模仿 Rust 的方法：它只要求可以调用一个特定方法，产生一个特定类型。

```rs
#include <concepts>

template <typename T>
concept shape = requires(const T &t) {
  { t.area() } -> std::same_as<double>;
};

template <shape T>
double twiceArea(const T &shape) {
  return shape.area() * 2.0;
} 
```

在 C++中，与上述 Rust 程序更接近的等效方法是使用抽象类和概念的组合。

```rs
#include <concepts>
#include <iostream>

struct Shape {
  Shape() {}
  virtual ~Shape() {}
  virtual double area() const = 0;
};

template <typename T>
concept shape = std::derived_from<T, Shape>;

struct Triangle : public Shape {
  double base;
  double height;

  Triangle(double base, double height)
      : base(base), height(height) {}

  // still will be used with static dispatch
  double area() const override {
    return 0.5 * base * height;
  }
};

template <shape T>
double twiceArea(const T &shape) {
  return shape.area() * 2;
}

int main() {
  Triangle triangle{1.0, 1.0};

  std::cout << twiceArea(triangle) << std::endl;
  return 0;
} 
```

然而，这仍然不同，因为概念只对模板的使用提出了要求，而不是对模板内`T`类型值的使用的限制。在 Rust 中，特质的界限约束了这两者。因此，以下代码在 C++中仍然可以编译。

```rs
#include <concepts>

struct Shape {
  Shape() {}
  virtual ~Shape() {}
  virtual double area() = 0;
};

template <typename T>
concept shape = std::derived_from<T, Shape>;

template <shape T>
double twiceArea(const T &shape) {
  // note the call to a method not defined in Shape
  return shape.volume() * 2;
} 
```

然而，在 Rust 中，等效的代码无法编译，反而会产生错误。

```rs
trait Shape {
    fn area(&self) -> f64;
}

fn twice_area<T: Shape>(shape: &T) -> f64 {
    // note the call to a method not defined in Shape
    2.0 * shape.volume()
}
```

```rs
error[E0599]: no method named `volume` found for reference `&T` in the current scope
 --> example.rs:7:17
  |
7 |     2.0 * shape.volume()
  |                 ^^^^^^ method not found in `&T` 
```

这些额外的静态检查意味着在许多情况下，当 C++模板很有用但难以正确实现时，Rust 泛型可以自由使用。

## 所需特性和人体工程学

在上述示例中，要求特质的函数是使用具有单独要求`T`是`Shape`的泛型类型`T`定义的，如下所示：

```rs
template <shape T>
double twiceArea(const T &shape) {
  return 2.0 * shape.area();
} 
```

```rs
fn twice_area<T: Shape>(shape: &T) -> f64 {
    2.0 * shape.area()
}
```

这些语法都是`requires`子句（C++）或`where`子句（Rust）的常见简写：

```rs
template <typename T>
  requires shape<T>
double twiceArea(const T &shape) {
  return 2.0 * shape.area();
} 
```

```rs
fn twice_area<T>(shape: &T) -> f64
where
    T: Shape,
{
    2.0 * shape.area()
}
```

当存在许多类型参数或这些类型参数必须实现许多特性时，更冗长的形式是首选。在某些情况下，还可以使用更简短的`impl`关键字。

```rs
double twiceArea(const shape auto &shape) {
  return 2.0 * shape.area();
} 
```

```rs
fn twice_area(shape: &impl Shape) -> f64 {
    2.0 * shape.area()
}
```

## 泛型和生命周期

当在 C++中定义一个使用类型模板参数的模板时，程序员必须手动跟踪存储在该类型对象中的引用的生命周期。

以下（人为设计的）C++示例可以无错误编译，但可能会以导致未定义行为的方式使用。

```rs
#include <memory>   struct Shape {
 Shape() {} virtual ~Shape() {} virtual double area() = 0; };   template<typename S>
void store(S s, std::unique_ptr<Shape> data) {
    // Will pointers or references in `s` become dangling while `data`
    // is still in use?
	*data = s;
} 
```

Rust 检查类型参数中包含的引用的生命周期界限。就像特质的对象类型一样，这些界限通常根据[生命周期省略规则](https://doc.rust-lang.org/reference/lifetime-elision.html)推断。当它们无法推断或推断错误时，可以手动声明界限。

在上述示例的 Rust 翻译中，必须手动给出生命周期限制，因为推断的限制是不正确的。没有显式限制，编译器将产生错误。

```rs
trait Shape {}   fn store<S: Shape>(x: S, data: &mut Box<dyn Shape>) {
    *data = Box::new(x);
}
```

```rs
error[E0310]: the parameter type `S` may not live long enough
 --> example.rs:7:5
  |
7 |     *data = Box::new(x);
  |     ^^^^^
  |     |
  |     the parameter type `S` must be valid for the static lifetime...
  |     ...so that the type `S` will meet its required lifetime bounds
  | 
```

当显式地给出推断的生命周期限制时，错误信息变得更加清晰。对于给定的`store`类型，`x`的参数可能是一个生命周期比盒子内容中的生命周期短的类型。

```rs
trait Shape {}   struct Triangle {
 base: f64, height: f64, }   impl Shape for Triangle {}   // The type parameter S is assigned no lifetime bound.
fn store<'a, S: Shape>(
    x: S,
    // The reference is assigned a fresh lifetime by rule
    // [lifetime-elision.function.implicit-lifetime-parameters].
    //
    // The trait object is assigned 'static by rule
    // [lifetime-elision.trait-object.default] and
    // [lifetime-elision.trait-object.innermost-type].
    data: &'a mut Box<dyn Shape + 'static>,
) {
    *data = Box::new(x);
}

// An example of how the implementation of store could be misused with
// the given type.
fn main() {
    let triangle = Triangle {
        base: 1.0,
        height: 2.0,
    };
    let mut b: Box<dyn Shape> = Box::new(triangle);
    {
        let short_lived_triangle = Triangle {
            base: 5.0,
            height: 10.0,
        };
        store(short_lived_triangle, &mut b);
    }
    // Here b contains a dangling reference.
}
```

对于这个特定的情况，最一般的解决方案是定义一个新的生命周期参数来限制`S`和`dyn Shape`。对于引用的类型参数可以省略，因为它将被分配一个新的生命周期参数。

```rs

#![allow(unused)] fn main() { trait Shape {} 

// 注意共同的边界

// -----------------here-\

// ----------------------|---------------------------and here-\

//                       v                                    v

fn store<'s, S: Shape + 's>(x: S, data: &mut Box<dyn Shape + 's>) {

    *data = Box::new(x);

}

}

```

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Concepts, interfaces, and static dispatch)
