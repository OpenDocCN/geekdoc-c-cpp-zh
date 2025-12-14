# 标签联合和 std::variant

> 原文：[`cel.cs.brown.edu/crp/idioms/data_modeling/tagged_unions.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/tagged_unions.html)

## C 风格的标签联合

因为在 C++ 中联合不能用于类型欺骗，所以它们通常与标签一起使用，以区分联合的哪个变体是活动的。

Rust 的联合类型等价物总是带标签的。它们是 Rust 枚举的泛化，枚举变体可以关联额外的数据。

```rs
enum Tag { Rectangle, Triangle };

struct Shape {
  Tag tag;
  union {
    struct {
      double width;
      double height;
    } rectangle;
    struct {
      double base;
      double height;
    } triangle;
  };

  double area() {
    switch (this->tag) {
    case Rectangle: {
      return this->rectangle.width *
             this->rectangle.height;
    }
    case Triangle: {
      return 0.5 * this->triangle.base *
             this->triangle.height;
    }
    }
  }
}; 
```

```rs

#![allow(unused)] fn main() { enum Shape {

    Rectangle { width: f64, height: f64 },

    Triangle { base: f64, height: f64 },

}

impl Shape {

    fn area(&self) -> f64 {

        match self {

            Shape::Rectangle {

                width,

                height,

            } => width * height,

            Shape::Triangle { base, height } => {

                0.5 * base * height

            }

        }

    }

}

}

```

当匹配枚举时，Rust 要求处理枚举的所有变体。在 C++ 标签 `switch` 上使用 `default` 的情况下，Rust 的 `match` 中可以使用通配符。

```rs
#include <iostream>   enum Tag { Rectangle, Triangle, Circle };   struct Shape {
 Tag tag; union { struct { double width; double height; } rectangle; struct { double base; double height; } triangle; struct { double radius; } circle; };    void print_shape() {
    switch (this->tag) {
    case Rectangle: {
      std::cout << "Rectangle" << std::endl;
      break;
    }
    default: {
      std::cout << "Some other shape"
                << std::endl;
      break;
    }
    }
  }
}; 
```

```rs

#![allow(unused)] fn main() { enum Shape {

Rectangle { width: f64, height: f64 }, Triangle { base: f64, height: f64 }, }   impl Shape {

    fn print_shape(&self) {

        match self {

            Shape::Rectangle { .. } => {

                println!("Rectangle");

            }

            _ => {

                println!("Some other shape");

            }

        }

    }

}

}

```

Rust 不支持 C++ 风格的 fallthrough，其中可以在转到下一个情况之前执行某些行为。然而，在 Rust 中，只要同时匹配的枚举变体绑定相同的名称和类型，就可以同时匹配多个枚举变体。

```rs

#![allow(unused)] fn main() { enum Shape {

Rectangle { width: f64, height: f64 }, Triangle { base: f64, height: f64 }, }   impl Shape {

    fn bounding_area(&self) -> f64 {

        match self {

            Shape::Rectangle { height, width }

            }

                height,

                base: width,

            } => width * height,

        }

    }

}

}

```

## 访问值而不检查判别符

与 C 风格的联合不同，Rust 总是在访问值之前要求匹配判别符。如果变体已经已知，例如，由于早期的检查，则通常可以将代码重构为在类型中编码知识，从而省略第二次检查（以及相应的错误处理）。

类似于以下 C++ 程序的程序需要在 Rust 中对类型进行更多重构才能达到相同的目标。

对应的 Rust 程序需要为 `Shape` 枚举的每个变体定义单独的类型，以便通过使用 `Triangle` 数组而不是 `Shape` 数组来在类型系统中表达所有值都是给定类型的事实。

```rs
#include <ranges>
#include <vector>

// Uses the same Shape definition.
enum Tag { Rectangle, Triangle };

struct Shape {
  Tag tag;
  union {
    struct {
      double width;
      double height;
    } rectangle;
    struct {
      double base;
      double height;
    } triangle;
  };
};

std::vector<Shape> get_shapes() {
  return std::vector<Shape>{
      Shape{Triangle, {.triangle = {1.0, 1.0}}},
      Shape{Triangle, {.triangle = {1.0, 1.0}}},
      Shape{Rectangle, {.rectangle = {1.0, 1.0}}},
  };
}

std::vector<Shape> get_shapes();

int main() {
  std::vector<Shape> shapes = get_shapes();

  auto is_triangle = [](Shape shape) {
    return shape.tag == Triangle;
  };

  // Create an iterator that only sees the
  // triangles. (std::views::filter is from C++20,
  // but the same effect can be acheived with a
  // custom iterator.)
  auto triangles =
      shapes | std::views::filter(is_triangle);

  double total_base = 0.0;
  for (auto &triangle : triangles) {
    // Skip checking the tag because we know we
    // have only triangles.
    total_base += triangle.triangle.base;
  }

  return 0;
} 
```

```rs

// 为每个变体定义一个单独的结构体。

struct Rectangle { width: f64, height: f64 }

struct  Triangle { base: f64, height: f64 }

enum Shape {

    Rectangle(Rectangle),

    Triangle(Triangle),

}

fn get_shapes() -> Vec<Shape> {

    vec![

        Shape::Triangle(Triangle {

            base: 1.0,

            height: 1.0,

        }),

        Shape::Triangle(Triangle {

            base: 1.0,

            height: 1.0,

        }),

        Shape::Rectangle(Rectangle {

            width: 1.0,

            height: 1.0,

        }),

    ]

}

fn main() {

    let shapes = get_shapes();

    // 这个迭代器仅遍历三角形

    // 通过迭代

    // 使用三角形类型而不是形状类型。

    let triangles = shapes

        .iter()

        // 仅保留三角形

        .filter_map(|shape| match shape {

            Shape::Triangle(t) => Some(t),

            _ => None,

        });

    let mut total_base = 0.0;

    for triangle in triangles {

        // 因为迭代器产生三角形

        // 而不是 Shapes，可以直接访问 base

        // 直接访问。

        total_base += triangle.base;

    }

}

```

在 Rust 中，这种用法很常见，变体通常从一开始就被设计成具有自己的类型。

在 C++ 中，这种方法也是可能的。它在 C++17 或更高版本中通常与 `std::variant` 一起使用。

## `std::variant` (since C++17)

当在 C++17 标准下编程时，`std::variant` 可以用来以更接近 Rust 枚举的方式表示标签联合。

```rs
#include <variant>

struct Rectangle {
  double width;
  double height;
};

struct Triangle {
  double base;
  double height;
};

using Shape = std::variant<Rectangle, Triangle>;

double area(const Shape &shape) {
  return std::visit(
      [](auto &&arg) -> double {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, Rectangle>) {
          return arg.width * arg.height;
        } else if constexpr (std::is_same_v<T, Triangle>) {
          return 0.5 * arg.base * arg.height;
        }
      },
      shape);
} 
```

因为 Rust 不依赖于模板来实现这种语言特性，所以当遗漏或添加新的标签联合变体时，错误信息更容易阅读，这消除了使用标签联合的障碍之一。比较 C++（使用 gcc）和 Rust 在省略 `Triangle` 情况下的错误。

The following two programs have the same error: each fails to handle a case of `Shape`.

```rs
#include <variant>

struct Rectangle {
  double width;
  double height;
};

struct Triangle {
  double base;
  double height;
};

using Shape = std::variant<Rectangle, Triangle>;

double area(const Shape &shape) {
  return std::visit(
      [](auto &&arg) -> double {
        using T = std::decay_t<decltype(arg)>;
        if constexpr (std::is_same_v<T, Rectangle>) {
          return arg.width * arg.height;
        }
      },
      shape);
} 
```

```rs
enum Shape {
    Rectangle { width: f64, height: f64 },
    Triangle { base: f64, height: f64 },
}

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Rectangle {
                width,
                height,
            } => width * height,
        }
    }
}
```

然而，错误信息差异很大。

```rs
example.cc: In instantiation of ‘area(const Shape&)::<lambda(auto:27&&)> [with auto:27 = const Triangle&]’:
/usr/include/c++/14.2.1/bits/invoke.h:61:36:   required from ‘constexpr _Res std::__invoke_impl(__invoke_other, _Fn&&, _Args&& ...) [with _Res = double; _Fn = area(const Shape&)::<lambda(auto:27&&)>; _Args = {const Triangle&}]’
   61 |     { return std::forward<_Fn>(__f)(std::forward<_Args>(__args)...); }
      |              ~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/bits/invoke.h:96:40:   required from ‘constexpr typename std::__invoke_result<_Functor, _ArgTypes>::type std::__invoke(_Callable&&, _Args&& ...) [with _Callable = area(const Shape&)::<lambda(auto:27&&)>; _Args = {const Triangle&}; typename __invoke_result<_Functor, _ArgTypes>::type = double]’
   96 |       return std::__invoke_impl<__type>(__tag{}, std::forward<_Callable>(__fn),
      |              ~~~~~~~~~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   97 |                                         std::forward<_Args>(__args)...);
      |                                         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/variant:1060:24:   required from ‘static constexpr decltype(auto) std::__detail::__variant::__gen_vtable_impl<std::__detail::__variant::_Multi_array<_Result_type (*)(_Visitor, _Variants ...)>, std::integer_sequence<long unsigned int, __indices ...> >::__visit_invoke(_Visitor&&, _Variants ...) [with _Result_type = std::__detail::__variant::__deduce_visit_result<double>; _Visitor = area(const Shape&)::<lambda(auto:27&&)>&&; _Variants = {const std::variant<Rectangle, Triangle>&}; long unsigned int ...__indices = {1}]’
 1060 |           return std::__invoke(std::forward<_Visitor>(__visitor),
      |                  ~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 1061 |               __element_by_index_or_cookie<__indices>(
      |               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 1062 |                 std::forward<_Variants>(__vars))...);
      |                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/variant:1820:5:   required from ‘constexpr decltype(auto) std::__do_visit(_Visitor&&, _Variants&& ...) [with _Result_type = __detail::__variant::__deduce_visit_result<double>; _Visitor = area(const Shape&)::<lambda(auto:27&&)>; _Variants = {const variant<Rectangle, Triangle>&}]’
 1820 |                   _GLIBCXX_VISIT_CASE(1)
      |                   ^~~~~~~~~~~~~~~~~~~
/usr/include/c++/14.2.1/variant:1882:34:   required from ‘constexpr std::__detail::__variant::__visit_result_t<_Visitor, _Variants ...> std::visit(_Visitor&&, _Variants&& ...) [with _Visitor = area(const Shape&)::<lambda(auto:27&&)>; _Variants = {const variant<Rectangle, Triangle>&}; __detail::__variant::__visit_result_t<_Visitor, _Variants ...> = double]’
 1882 |             return std::__do_visit<_Tag>(
      |                    ~~~~~~~~~~~~~~~~~~~~~^
 1883 |               std::forward<_Visitor>(__visitor),
      |               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 1884 |               static_cast<_Vp>(__variants)...);
      |               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
example.cc:17:20:   required from here
   17 |   return std::visit(
      |          ~~~~~~~~~~^
   18 |       [](auto &&arg) -> double {
      |       ~~~~~~~~~~~~~~~~~~~~~~~~~~
   19 |         using T = std::decay_t<decltype(arg)>;
      |         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   20 |         if constexpr (std::is_same_v<T, Rectangle>) {
      |         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   21 |           return arg.width * arg.height;
      |           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   22 |         }
      |         ~
   23 |       },
      |       ~~
   24 |       shape);
      |       ~~~~~~
example.cc:23:7: error: no return statement in ‘constexpr’ function returning non-void
   23 |       },
      |       ^
example.cc: In lambda function:
example.cc:23:7: warning: control reaches end of non-void function [-Wreturn-type] 
```

```rs
error[E0004]: non-exhaustive patterns: `&Shape::Triangle { .. }` not covered
 --> example.rs:8:15
  |
8 |         match self {
  |               ^^^^ pattern `&Shape::Triangle { .. }` not covered
  |
note: `Shape` defined here
 --> example.rs:1:6
  |
1 | enum Shape {
  |      ^^^^^
2 |     Rectangle { width: f64, height: f64 },
3 |     Triangle { base: f64, height: f64 },
  |     -------- not covered
  = note: the matched value is of type `&Shape`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
12~             } => width * height,
13~             &Shape::Triangle { .. } => todo!(),
  | 
```

## 使用不安全的 Rust 来避免检查判别符

在无法重写代码以使用 上述方法 的情况下，仍然可以检查判别符，然后使用 [`unreachable!` 宏](https://doc.rust-lang.org/std/macro.unreachable.html) 来避免处理不可能的情况。然而，这仍然涉及到实际检查判别符。如果必须避免检查判别符的成本，则可以使用 [不安全的函数 `unreachable_unchecked`](https://doc.rust-lang.org/std/hint/fn.unreachable_unchecked.html) 来避免处理该情况，并指示编译器优化器应假设该情况无法到达，从而可以优化掉判别符检查。

与在 C++ 示例中访问非活动变体是未定义行为类似，到达 `unreachable_unchecked` 也是未定义行为。与任何基于 `unsafe` 的性能优化一样，您始终应该首先测量安全检查的性能影响，并且只有在绝对必要时才求助于 `unsafe` 代码。

```rs

enum Shape {

Rectangle { width: f64, height: f64 }, Triangle { base: f64, height: f64 }, }   impl Shape {

fn area(&self) -> f64 { match self { Shape::Rectangle { width, height, } => width * height, Shape::Triangle { base, height } => { 0.5 * base * height } } } }   fn get_triangles() -> Vec<Shape> {

vec![ Shape::Triangle { base: 1.0, height: 1.0, }, Shape::Triangle { base: 1.0, height: 1.0, }, ] }   use std::hint::unreachable_unchecked;

fn main() {

    let mut total_base = 0.0;

    for triangle in get_triangles() {

        let Shape::Triangle { base, .. } = triangle else {

            // 安全性：get_triangles 确保会生成三角形，所以

            // 其他情况无法到达。

            unsafe { unreachable_unchecked() }

        };

        total_base += base;

    }

}

```

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Tagged unions and std::variant)
