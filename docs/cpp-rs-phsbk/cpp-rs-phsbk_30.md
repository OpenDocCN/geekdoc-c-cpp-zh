# 类型提升和转换

> 原文：[`cel.cs.brown.edu/crp/idioms/promotions_and_conversions.html`](https://cel.cs.brown.edu/crp/idioms/promotions_and_conversions.html)

## 左值到右值

在 C++ 中，当需要时，左值会自动转换为右值。

在 Rust 中，左值的等效是“位置表达式”（表示内存位置的表达式），而右值的等效是“值表达式”。当需要时，位置表达式会自动转换为值表达式。

```rs
int main() {
  // Local variables are lvalues,
  int x(0);
  // and therefore may be assigned to.
  x = 42;

  // x is converted to an rvalue when needed.
  int y = x + 1;
} 
```

```rs

fn main() {

    // 局部变量是位置表达式，

    let mut x = 0;

    // 因此可以被赋值。

    x = 42;

    // 当需要时，x 被转换为值表达式

    // 需要时。

    let y = x + 1;

}

```

## 数组到指针

在 C++ 中，数组根据需要自动转换为指针。

在 Rust 中与此等效的是向量引用和数组引用自动转换为切片引用。

```rs
#include <cstring>

int main() {
  char example[6] = "hello";
  char other[6];

  // strncpy takes arguments of type char*
  strncpy(other, example, 6);
} 
```

```rs

fn third(ts: &[char]) -> Option<&char> {

    ts.get(2)

}

fn main() {

    let vec: Vec<char> = vec!['a', 'b', 'c'];

    let arr: [char; 3] = ['a', 'b', 'c'];

    third(&vec);

    third(&arr);

}

```

因为切片引用可以以内存安全的方式轻松使用，所以在 Rust 中通常建议用切片引用来定义函数，而不是用向量的引用或数组的引用，除非需要特定的向量或数组功能。

与 C++ 中的数组到指针的转换内置于语言不同，这实际上是由 `Deref` 特性提供的通用机制，该特性提供了一种用户定义的转换。

## 函数到指针

在 C++ 中，函数和静态成员函数会自动转换为函数指针。

Rust 执行相同的转换。除了不接收 `self` 参数的函数和成员之外，构造函数（正确构造函数）也有函数类型，并且可以转换为函数指针。非捕获闭包没有函数类型，但也可以转换为函数指针。

```rs
int twice(int n) {
  return n * n;
}

struct MyPair {
  int x;
  int y;

  MyPair(int x, int y) : x(x), y(y) {}

  static MyPair make() {
    return MyPair{0, 0};
  }
};

int main() {
  // convert a function to a function pointer
  int (*twicePtr)(int) = twice;
  int result = twicePtr(5);

  // Per C++23 11.4.5.1.6, can't take the address
  // of a constructor.
  // MyPair (*ctor)(int, int) = MyPair::MyPair;
  // MyPair pair = ctor(10, 20);

  // convert a static method to a function
  // pointer
  MyPair (*methodPtr)() = MyPair::make;
  MyPair pair2 = methodPtr();

  // convert a non-capturing closure to a
  // function pointer
  int (*closure)(int) = [](int x) -> int {
    return x * 5;
  };
  int closureRes = closure(2);
} 
```

```rs

fn twice(x: i32) -> i32 {

    x * x

}

struct MyPair(i32, i32);

impl MyPair {

    fn new() -> MyPair {

        MyPair(0, 0)

    }

}

fn main() {

    // 将函数转换为函数指针

    let twicePtr: fn(i32) -> i32 = twice;

    let res = twicePtr(5);

    // 将构造函数转换为函数指针

    let ctorPtr: fn(i32, i32) -> MyPair = MyPair;

    let pair = ctorPtr(10, 20);

    // 将静态方法转换为函数

    // 指针

    let methodPtr: fn() -> MyPair = MyPair::new;

    let pair2 = methodPtr();

    // 将非捕获闭包转换为函数

    // 函数指针

    let closure: fn(i32) -> i32 = |x: i32| x * 5;

    let closureRes = closure(2);

}

```

## 数值提升和数值转换

在 C++ 中，数值类型之间存在几种隐式转换。最常见的是数值提升，它将数值类型转换为更大的类型。

这些无损转换在 Rust 中不是隐式的。相反，必须显式使用 `Into::into()` 方法来执行这些转换。这些转换由 `From` 和 `Into` 特性的实现提供。Rust 标准库提供的转换列表可以在该特性的文档页面（[文档页面](https://doc.rust-lang.org/std/convert/trait.From.html#implementors)）上找到。

```rs
int main() {
  int x(42);
  long y = x;

  float a(1.0);
  double b = a;
} 
```

```rs

fn main() {

    let x: i32 = 42;

    let y: i64 = x.into();

    let a: f32 = 1.0;

    let b: f64 = a.into();

}

```

C++ 中存在一些不是无损的隐式转换。例如，整数可以隐式转换为无符号整数。

在 Rust 中，这些转换也必须是显式的，并由 `TryFrom` 和 `TryInto` 特性提供，这些特性要求处理值不映射到其他类型的情况。

```rs
int main() {
  int x(42);
  unsigned int y(x);

  float a(1.0);
  double b(a);
} 
```

```rs

use std::convert::TryInto;

fn main() {

    let x: i32 = 42;

    let y: u32 = match x.try_into() {

        Ok(x) => x,

        Err(err) => {

            panic!("无法转换！ {:?}", err);

        }

    };

}

```

在 C++ 中，一些转换既不支持 `From` 也不支持 `TryFrom`，因为不存在明确的转换选择，或者因为它们不是值保持的。例如，在 C++ 中，`int32_t` 可以隐式转换为 `float`，尽管 `float` 无法精确表示所有 32 位整数，但在 Rust 中没有为 `f32` 实现 `TryFrom<i32>`。

在 Rust 中，从 `i32` 转换到 `f32` 的唯一方法是使用 `as` 操作符（[`as` 操作符](https://doc.rust-lang.org/stable/reference/expressions/operator-expr.html#r-expr.as.coercions)）。实际上，该操作符可以用于在其它原始类型之间进行转换，并且不会引发 panic 或产生未定义的行为，但它可能不会按照预期的方式转换（例如，它可能使用与预期不同的舍入模式，或者它可能截断而不是饱和）。

```rs
#include <cstdint>

int main() {
  int32_t x(42);
  float a = x;
} 
```

```rs

fn main() {

    let x: i32 = 42;

    let a: f32 = x as f32;

}

```

### `isize` 和 `usize`

在 Rust 标准库中，`isize` 和 `usize` 类型用于表示索引值（类似于 C++ 中的 `size_t`）。然而，通常不鼓励用于其他目的，而是使用显式大小的类型，如 `u32`。这导致了一个情况，即类型为 `u32` 的值必须转换为 `usize` 以用于索引，但 `Into<usize>` 没有为 `u32` 实现。

在这种情况下，最佳实践是使用 `TryInto`，如果不需要进一步处理失败原因的错误处理，则调用 `unwrap`，在转换点引发 panic。

这是一种更受欢迎的方法，因为它可以防止继续使用错误值的可能性。例如，考虑使用`as`将`u64`转换为具有 32 位表示的`usize`，这会截断结果。比`u32::MAX`大 1 的值将截断为`0`，这可能会在数据结构中成功检索到错误值，从而掩盖错误并产生意外的行为。

### 枚举

在 C++中，枚举可以隐式转换为整数类型。

在 Rust 中，转换需要使用`as`运算符，并且建议提供`From`和`TryFrom`实现，以便在枚举及其表示类型之间来回移动。示例和更多细节在枚举章节中给出。

## 资格转换

在 C++中，资格转换允许在不需要资格转换的情况下使用 const（或 volatile）值。

在 Rust 中，等效的转换允许在期望非`mut`变量或引用的地方使用`mut`变量和`mut`引用。

```rs
#include <iostream>
#include <string>

void display(const std::string &msg) {
  std::cout << "Displaying: " << msg << std::endl;
}

int main() {
  // no const qualifier
  std::string message("hello world");

  // used where const expected
  display(message);
} 
```

```rs

fn display(msg: &str) {

    println!("{}", msg);

}

fn main() {

    let mut s: String = "hello world".to_string();

    let message: &mut str = s.as_mut();

    display(message);

}

```

## 整数字面量

在 C++中，没有后缀表示类型的整数字面量具有从`int`、`long int`或`long long int`中可以容纳的最小类型。当字面量随后被分配给不同类型的变量时，将执行隐式转换。

在 Rust 中，整数字面量的类型取决于上下文。当没有足够的信息来推断类型时，默认假设为`i32`，或者可能需要提供一些类型注解。

```rs
#include <cstdint>
#include <iostream>

int main() {
  // Compiles without error (but with a warning).
  uint32_t x = 4294967296;

  // assumes int
  auto y = 1;

  // literal is given a larger type, so it prints
  // correctly
  std::cout << 4294967296 << std::endl;

  // these work as expected
  std::cout << INT64_C(4294967296) << std::endl;

  uint64_t z = INT64_C(4294967296);
  std::cout << z << std::endl;
} 
```

```rs

fn main() {

    // 错误：字面量超出`u32`的范围

    // let x: u32 = 4294967296;

    // 假设 i32

    let y = 1;

    // 编译失败，因为它被推断为 i32

    // print!("{}", 4294967296);

    // 这些是可行的。

    println!("{}", 4294967296u64);

    let z: u64 = 4294967296;

    println!("{}", z);

}

```

## 安全的布尔值

安全布尔值习语的存在是为了使类型作为条件使用成为可能。自 C++11 以来，这个习语变得容易实现。

在 Rust 中，而不是将值转换为布尔值，通常使用`match`、`if let`或`let else`来匹配值。根据情况，可能使用的匹配机制是`match`、`if let`或`let else`。

```rs
struct Wire {
  bool ready;
  unsigned int value;

  explicit operator bool() const { return ready; }
};

int main() {
  Wire w{false, 0};
  // ...

  if (w) {
    // use w.value
  } else {
    // do something else
  }
} 
```

```rs

enum Wire {

    Ready(u32),

    NotReady,

}

fn main() {

    let wire = Wire::NotReady;

    // ...

    // match

    match wire {

        Wire::Ready(v) => {

            // use value v

        }

        Wire::NotReady => {

            // 执行其他操作

        }

    }

    // if let

    if let Wire::Ready(v) = wire {

        // use value v

    }

    // let else

    let Wire::Ready(v) = wire else {

        // 执行一些不会继续的操作，

        // 类似于早期返回

        return;

    };

}

```

## 用户定义的转换

用户定义的转换将在单独的章节中介绍。

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Type promotions and conversions)
