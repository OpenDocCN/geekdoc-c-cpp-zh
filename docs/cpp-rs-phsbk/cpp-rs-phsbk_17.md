# 哨兵值

> 原文：[`cel.cs.brown.edu/crp/idioms/null/sentinel_values.html`](https://cel.cs.brown.edu/crp/idioms/null/sentinel_values.html)

哨兵值是在带内值，表示特殊的情况，例如在迭代器中到达有效数据的末尾。

## `nullptr`

许多 C++ 中的设计借鉴了 C 的约定，即使用空指针作为返回拥有指针的方法的哨兵值。例如，一个解析大型结构的方法在失败的情况下可能会产生 `nullptr`。

在 Rust 中类似的情况会使用类型 `Option<Box<LargeStructure>>`。

```rs
#include <memory>

class LargeStructure {
  int field;
  // many fields ...
};

std::unique_ptr<LargeStructure>
parse(char *data, size_t len) {
  // ...

  // on failure
  return nullptr;
} 
```

```rs

#![allow(unused)] fn main() { struct LargeStructure {

    field: i32,

    // 许多字段 ...

}

fn parse(

    data: &[u8],

) -> Option<Box<LargeStructure>> {

    // ...

    // 失败时

    None

}

}

```

`Box<T>` 类型在表示堆上某个 `T` 的唯一所有者指针方面与 `std::unique_ptr<T>` 具有相同的意义，但与 `std::unique_ptr` 不同，它不能为空。当与 `Box<T>` 结合使用时，Rust 的 `Option<T>`（在 C++ 中类似于 `std::optional<T>`）可以表示可选指针，例如 `Optional<Box<T>>`。在这些情况（以及一些其他情况）中，编译器利用 `Box` 不能为空的事实，将表示优化为与 `Box<T>` 相同的大小。

在 Rust 中，也常见为了在运行时提供失败原因而支付额外字节的成本，使用返回类型为 `Result<T, E>`（类似于 C++23 中的 `std::expected`）。

## 整数哨兵

当一个可能失败的功能产生一个整数时，也常见使用其他未使用或不太可能的整数值作为哨兵值，例如 `0` 或 `INT_MAX`。

在 Rust 中，`Option` 类型用于此目的。在零值确实无法产生的情况下，例如下面的 GCD 算法，可以使用类型 `NonZero<T>` 来表示这一点。与 `Option<Box<T>>` 一样，编译器优化表示以利用未使用的值（在这种情况下为 `0`），以确保 `Option<NonZero<T>>` 的表示与 `Option<T>` 的表示相同。

```rs
#include <algorithm>

int gcd(int a, int b) {
  if (b == 0 || a == 0) {
    // returns 0 to indicate invalid input
    return 0;
  }

  while (b != 0) {
    int temp = b;
    b = a % b;
    a = temp;
  }
  return std::abs(a);
} 
```

```rs

use std::num::NonZero;

fn gcd(

    mut a: i32,

    mut b: i32,

) -> Option<NonZero<i32>> {

    如果 a 等于 0 或 b 等于 0 {

        return None;

    }

    while b != 0 {

        let temp = b;

        b = a % b;

        a = temp;

    }

    // 在这一点上，a 保证不会是

    // 零。来自 `NonZero::new` 的 `Some` 情况

    // 与 `Some` 的意义不同

    // 从此函数返回，但在这里

    // 碰巧一致。

    NonZero::new(a.abs())

}

fn main() {

assert!(gcd(5, 0) == None); assert!(gcd(0, 5) == None); assert!(gcd(5, 1) == NonZero::new(1)); assert!(gcd(1, 5) == NonZero::new(1)); assert!(gcd(2 * 2 * 3 * 5 * 7, 2 * 2 * 7 * 11) == NonZero::new(2 * 2 * 7)); assert!(gcd(2 * 2 * 7 * 11, 2 * 2 * 3 * 5 * 7) == NonZero::new(2 * 2 * 7)); }

```

作为旁注，也可以通过在整个算法中保留非零属性来避免在末尾进行冗余的零检查，并且无需使用不安全的 Rust。

```rs

use std::num::NonZero;

fn gcd(x: i32, mut b: i32) -> Option<NonZero<i32>> {

    if b == 0 {

        return None;

    }

    // a 是保证非零的，所以我们记录这个事实在 a 的类型中。

    let mut a = NonZero::new(x)?;

    while let Some(temp) = NonZero::new(b) {

        b = a.get() % b;

        a = temp;

    }

    Some(a.abs())

}

fn main() {

assert!(gcd(5, 0) == None); assert!(gcd(0, 5) == None); assert!(gcd(5, 1) == NonZero::new(1)); assert!(gcd(1, 5) == NonZero::new(1)); assert!(gcd(2 * 2 * 3 * 5 * 7, 2 * 2 * 7 * 11) == NonZero::new(2 * 2 * 7)); assert!(gcd(2 * 2 * 7 * 11, 2 * 2 * 3 * 5 * 7) == NonZero::new(2 * 2 * 7)); }

```

## `std::optional`

在 C++中，`std::optional`用作哨兵值的情况下，Rust 中的`Option`可以用于相同的目的。这两者之间的主要区别在于，安全的 Rust 需要显式检查值是否为`None`，而在 C++中，可以尝试访问值而不进行检查（这可能导致未定义的行为）。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Sentinel values)
