# 奇怪地反复出现的模板模式（CRTP）

> 原文：[`cel.cs.brown.edu/crp/patterns/crtp.html`](https://cel.cs.brown.edu/crp/patterns/crtp.html)

C++ 中的奇怪地反复出现的模板模式（CRTP）用于在基类中定义的方法的定义中使派生类的具体类型可用。

## 使用静态多态共享实现

CRTP 的基本用途是减少使用静态多态的实现中的冗余。在这个用例中，`this` 指针被转换为模板参数提供的类型，以便可以调用派生类的方法。这使得基类中实现的方法可以调用派生类中的方法，而无需将它们声明为虚拟的，避免了动态分派的开销。

在下面的示例中，`Triangle` 和 `Square` 有一个共同的 `twiceArea` 实现，无需动态分派。这种用法在 Rust 中使用默认特质方法来解决。

```rs
#include <iostream>

template <typename T>
struct Shape {
  // This implementation is shared and can call
  // the area method from derived classes without
  // declaring it virtual.
  double twiceArea() {
    return 2.0 * static_cast<T *>(this)->area();
  }
};

struct Triangle : public Shape<Triangle> {
  double base;
  double height;

  Triangle(double base, double height)
      : base(base), height(height) {}

  double area() {
    return 0.5 * base * height;
  }
};

struct Square : public Shape<Square> {
  double side;

  Square(double side) : side(side) {}

  double area() {
    return side * side;
  }
};

int main() {
  Triangle triangle{2.0, 1.0};
  Square square{2.0};

  std::cout << triangle.twiceArea() << std::endl;
  std::cout << square.twiceArea() << std::endl;
} 
```

```rs

```

trait Shape {

    fn area(&self) -> f64;

    fn twice_area(&self) -> f64 {

        2.0 * self.area()

    }

}

struct Triangle {

    base: f64,

    height: f64,

}

impl Shape for Triangle {

    fn area(&self) -> f64 {

        0.5 * self.base * self.height

    }

}

struct Square {

    side: f64,

}

impl Shape for Square {

    fn area(&self) -> f64 {

        self.side * self.side

    }

}

fn main() {

    let triangle = Triangle {

        base: 2.0,

        height: 1.0,

    };

    let square = Square { side: 2.0 };

    println!("{}", triangle.twice_area());

    println!("{}", square.twice_area());

}

```rs

```

Rust 中默认方法调用 `area` 静态执行无需额外操作的原因是，在 Rust 中对 `self` 的方法调用始终是静态解析的。这是可能的，因为 Rust 中没有具体类型之间的继承。尽管默认方法是在特质中定义的，但实际上它是实现结构的一部分。

## 方法链

CRTP 的另一个常见用途是在基类提供了一个要链式调用的方法实现时实现方法链。

在 C++ 中，模板参数用于确保从共享函数返回的类型是派生类的类型，这样就可以在它上面调用派生类中定义的进一步方法。模板参数还用于在派生类型上调用方法，而无需将该方法声明为虚拟的。

在 Rust 中，模板参数不是必需的，因为 `Self` 类型在特质中可用，可以用来引用实现结构的类型。

```rs
#include <iostream>
#include <span>
#include <string>
#include <vector>

// D is the type of the derived class
template <typename D>
struct Combinable {
  D combineWith(D &d);

  // concat is implemented in the base class, but
  // operates on values of the derived class.
  D concat(std::span<D> vec) {
    D acc(*static_cast<D *>(this));

    for (D &v : vec) {
      acc = acc.combineWith(v);
    }

    return acc;
  }
};

struct Sum : Combinable<Sum> {
  int sum;

  Sum(int sum) : sum(sum) {}

  Sum combineWith(Sum s) {
    return Sum(sum + s.sum);
  }

  // Sum includes an additional method that can be
  // chained.
  Sum mult(int n) {
    return Sum(sum * n);
  }
};

int main() {
  Sum s(0);
  std::vector<Sum> v{1, 2, 3, 4};
  Sum x = s.concat(v)
              // Even though concat is part of the
              // base class, it returns a value of
              // the implementing class, making it
              // possible to chain methods
              // specific to that class.
              .mult(2)
              .combineWith(5);
  std::cout << x.sum << std::endl;
} 
```

```rs

```

// 不需要泛型类型：Self 已经

// 指的是实现类型。

trait Combinable {

    fn combine_with(&self, other: &Self) -> Self;

    // concat 有一个默认实现

    // 以 Self 的术语。

    fn concat(&self, others: &[Self]) -> Self

    where

        Self: Clone,

    {

        let mut acc = self.clone();

        for v in others {

            acc = acc.combine_with(v);

        }

        acc

    }

}

#[derive(Clone)]

struct Sum(i32);

impl Sum {

    // Sum 包含一个额外的

    // 链式

    fn mult(&self, n: i32) -> Self {

        Self(self.0 * n)

    }

}

impl Combinable for Sum {

    fn combine_with(&self, other: &Self) -> Self {

        Self(self.0 + other.0)

    }

}

fn main() {

    let s = Sum(0);

    let v = vec![Sum(1), Sum(2), Sum(3), Sum(4)];

    let x = s

        .concat(&v)

        // 即使 concat 是

        // 特性，它返回一个

        // 实现类型，使其

        // 可以链式调用特定类型的

        .mult(2)

        .combine_with(&Sum(5));

    println!("{}", x.0)

}

```rs

```

再次，`Self` 可以引用实现类型的原因是 Rust 不支持具体类型之间的继承。这与 C++ 相比，其中值可以在任何数量的具体类型中使用，因此不清楚 `Self` 应该引用哪种类型。

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Curiously recurring template pattern (CRTP))
