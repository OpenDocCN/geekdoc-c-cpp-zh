# 可选返回值

> 原文：[`cel.cs.brown.edu/crp/idioms/out_params/optional_return.html`](https://cel.cs.brown.edu/crp/idioms/out_params/optional_return.html)

C++中一个用于从方法或函数中可选产生结果的习惯用法是使用一个引用参数以及布尔或整数返回值来指示是否产生了结果。这可能是出于与使用 多返回值的输出参数 相同的原因：

+   与 C++11 之前的版本兼容，

+   在使用 C++风格的 C++代码库中工作，并且

+   性能问题。

Rust 在可选返回值方面采用的习惯用法是返回类型为 `Option` 的值。

```rs
#include <iostream>

bool safe_divide(unsigned int dividend,
                 unsigned int divisor,
                 unsigned int &quotient) {
  if (divisor != 0) {
    quotient = dividend / divisor;
    return true;
  } else {
    return false;
  }
}

void go(unsigned int dividend,
        unsigned int divisor) {
  unsigned int quotient;
  if (safe_divide(dividend, divisor, quotient)) {
    std::cout << quotient << std::endl;
  } else {
    std::cout << "Division failed!" << std::endl;
  }
}

int main() {
  go(10, 2);
  go(10, 0);
} 
```

```rs

```

fn safe_divide(

    dividend: u32,

    divisor: u32,

) -> Option<u32> {

    if divisor != 0 {

        Some(dividend / divisor)

    } else {

        None

    }

}

fn go(dividend: u32, divisor: u32) {

    match safe_divide(dividend, divisor) {

        Some(quotient) => {

            println!("{}", quotient);

        }

        None => {

            println!("Division failed!");

        }

    }

}

fn main() {

    go(10, 2);

    go(10, 0);

}

```rs

```

当在失败情况下有提供有用信息时，可以使用 `Result` 类型代替。错误处理的章节描述了 `Result` 的使用。

## 返回指针

当返回的值是一个指针时，C++中另一个常见的习惯用法是使用 `nullptr` 来表示可选情况。在 Rust 对该习惯用法的翻译中，也使用了 `Option`，以及像 `&` 或 `Box` 这样的引用类型。有关更多详细信息，请参阅关于使用 `nullptr` 作为哨兵值的章节。

## 直接转换的问题

将使用输出参数的原例转换为 Rust 是可能的，但生成的代码不符合习惯用法。

```rs

```

// 不符合 Rust 习惯用法

fn safe_divide(dividend: u32, divisor: u32, quotient: &mut u32) -> bool {

    if divisor != 0 {

        *quotient = dividend / divisor;

        true

    } else {

        false

    }

}

fn go(dividend: u32, divisor: u32) {

    let mut quotient: u32 = 0; // 初始化为任意值

    if safe_divide(dividend, divisor, &mut quotient) {

        println!("{}", quotient);

    } else {

        println!("Division failed!");

    }

}

fn main() {

    go(10, 2);

    go(10, 0);

}

```rs

```

这与使用输出参数进行 多返回值 存在相同的问题。

## 与 C++自 C++17 以来的相似之处

C++17 及以后的版本提供了 `std::optional`，它可以以类似于习惯用法 Rust 示例的方式表达可选返回值。

```rs
#include <iostream>
#include <optional>

std::optional<unsigned int> safe_divide(unsigned int dividend,
                                        unsigned int divisor) {
  if (divisor != 0) {
    return std::optional<unsigned int>(dividend / divisor);
  } else {
    return std::nullopt;
  }
}

void go(unsigned int dividend, unsigned int divisor) {
  if (auto quotient = safe_divide(dividend, divisor)) {
    std::cout << *quotient << std::endl;
  } else {
    std::cout << "Division failed!" << std::endl;
  }
}

int main() {
  go(10, 2);
  go(10, 0);
} 
```

## 有用的 `Option` 工具

Rust 为简化返回 `Option` 的函数的使用提供了几个语法糖。如果应该将失败传播给调用者，则使用 `?` 操作符：

```rs

```

#![allow(unused)] fn main() { fn safe_divide(dividend: u32, divisor: u32) -> Option<u32> {

if divisor != 0 { Some(dividend / divisor) } else { None } }   fn go(dividend: u32, divisor: u32) -> Option<()> {

    let quotient = safe_divide(dividend, divisor)?;

    println!("{}", quotient);

    Some(())

}

}

```rs

```

如果`None`不应该被传播，有时使用[let-else 语法](https://doc.rust-lang.org/rust-by-example/flow_control/let_else.html)会更清晰。

```rs

```

fn safe_divide(dividend: u32, divisor: u32) -> Option<u32> {

if divisor != 0 { Some(dividend / divisor) } else { None } }   fn go(dividend: u32, divisor: u32) {

    let Some(quotient) = safe_divide(dividend, divisor) else {

        println!("除法失败！");

        return;

    };

    println!("{}", quotient);

}

fn main() {

go(10, 2); go(10, 0); }

```rs

```

如果在 None 情况下应该使用默认值，可以使用以下方法：[`Option::unwrap_or`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or), [`Option::unwrap_or_else`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else), [`Option::unwrap_or_default`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default), 或 [`Option::unwrap`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap)。

```rs

```

fn safe_divide(dividend: u32, divisor: u32) -> Option<u32> {

if divisor != 0 { Some(dividend / divisor) } else { None } }   fn expensive_computation() -> u32 {

    // ...

0 }

fn go(dividend: u32, divisor: u32) {

    // 如果是 None，则返回给定的值。

    let result = safe_divide(dividend, divisor).unwrap_or(0);

    // 如果是 None，则返回调用给定函数的结果。

    let result2 = safe_divide(dividend, divisor).unwrap_or_else(expensive_computation);

    // 如果是 None，则返回 Default::default()，对于 u32 来说就是 0。

    let result3 = safe_divide(dividend, divisor).unwrap_or_default();

    // 如果是 None，则引发恐慌。建议使用其他方法！

    // let result3 = safe_divide(dividend, divisor).unwrap();

}

fn main() {

go(10, 2); go(10, 0); }

```rs

```

在性能敏感的代码中，如果你已经手动检查结果保证是`Some`，可以使用[`Option::unwrap_unchecked`](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_unchecked)，但这是一个不安全的方法。

有[额外的实用方法](https://doc.rust-lang.org/std/option/#boolean-operators)，这些方法可以简洁地处理`Option`值，本书在异常和错误处理章节中对此进行了介绍。

## 另一种方法

在 Rust 中，返回可选值的一种替代方法是要求函数的调用者证明他们传递给函数的值不会导致失败情况。

对于上述安全除法示例，这涉及到调用者保证提供的除数不为零。在以下示例中，这是通过动态检查来完成的。在其他上下文中，所需证据可能可以静态地获得，由更高层的调用者提供，或者被多次使用。在这些情况下，这种方法可以减少运行时成本和代码复杂性。

```rs

```

use std::convert::TryFrom;

use std::num::NonZero;

fn safe_divide(dividend: u32, divisor: NonZero<u32>) -> u32 {

    // 这更高效，因为跳过了溢出检查。

    dividend / divisor

}

fn go(dividend: u32, divisor: u32) {

    let Ok(safe_divisor) = NonZero::try_from(divisor) else {

        println!("不能除以！");

        return;

    };

    let quotient = safe_divide(dividend, safe_divisor);

    println!("{}", quotient);

}

fn main() {

    go(10, 2);

    go(10, 0);

}

```rs

```

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为我们提供关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Optional return values)
