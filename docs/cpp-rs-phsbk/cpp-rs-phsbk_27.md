# 预期错误

> 原文：[`cel.cs.brown.edu/crp/idioms/exceptions/expected_errors.html`](https://cel.cs.brown.edu/crp/idioms/exceptions/expected_errors.html)

在 C++ 中，`throw` 既能产生错误（抛出的异常）并启动非局部控制流（回滚到最近的 `catch` 块）。在 Rust 中，错误值（`Option::None` 或 `Result::Err`）作为正常值从函数返回。Rust 的 `return` 语句可以用来从函数中提前返回。

```rs
#include <stdexcept>

double divide(double dividend, double divisor) {
  if (divisor == 0.0) {
    throw std::domain_error("zero divisor");
  }

  return dividend / divisor;
} 
```

```rs

#![allow(unused)] fn main() { fn divide(

    dividend: f64,

    divisor: f64,

) -> Option<f64> {

    if divisor == 0.0 {

        return None;

    }

    Some(dividend / divisor)

}

}

```

需要返回类型表明可能发生错误的要求意味着允许有错误的回调需要提供 `Option` 或 `Result` 返回类型。省略这一点就像在 C++ 中要求回调为 `noexcept` 一样。不需要表明错误但将在允许错误的地方用作回调的函数需要将它们的结果包装在 `Option::Some` 或 `Result::Ok` 中。

```rs
#include <stdexcept>

int produce_42() {
  return 42;
}

int fail() {
  throw std::runtime_error("oops");
}

int useCallback(int (*func)(void)) {
  return func();
}

int main() {
  try {
    int x = useCallback(produce_42);
    int y = useCallback(fail);

    // use x and y
  } catch (std::runtime_error &e) {
    // handle error
  }
} 
```

```rs

fn produce_42() -> i32 {

    42

}

fn fail() -> Option<i32> {

    None

}

fn use_callback(

    f: impl Fn() -> Option<i32>,

) -> Option<i32> {

    f()

}

fn main() {

    // 需要包装 produce_42 以匹配

    // 预期类型

    let Some(x) =

        use_callback(|| Some(produce_42()))

    else {

        // 处理错误

        return;

    };

    let Some(y) = use_callback(fail) else {

        // 处理错误

        return;

    };

    // use x and y

}

```

## 处理错误

在 C++ 中，处理异常的唯一方法是 `catch`。在 Rust 中，可以使用 `Result` 和 `Option` 与所有处理 标记联合 的功能。最合适的方法取决于程序的目的。

在 Rust 中，处理由 `Result` 指示的错误的基本方法是使用 `match`。

使用 `match` 是最通用的方法，因为它可以显式处理额外的案例，并且可以用作表达式。`match` 表明所有分支具有同等重要性。

```rs
#include <vector>
#include <stdexcept>

int main() {
    std::vector<int> v;
    // ... populate v ...
    try {
        auto x = v.at(0);
        // use x
    } catch (std::out_of_range &e) {
        // handle error
    }
} 
```

```rs

fn main() {

    let mut v = Vec::<i32>::new();

    // ... populate v ...

    match v.get(0) {

        Some(x) => {

            // use x

        }

        None => {

            // 处理错误

        }

    }

}

```

因为只处理 Rust 枚举的一个变体非常常见，所以 `if let` 语法支持这种用法。该语法既清楚地表明只有一个案例很重要，又减少了缩进级别。

`if let` 比不上 `match` 通用。它也可以用作表达式，但只能区分一个案例与其他所有案例。`if let` 表明 `else` 情况不是正常情况，而是将发生某种默认处理或产生某个默认值。

注意，使用 `Result` 时，`if let` 不允许访问错误值。

```rs

fn main() {

    let mut v = Vec::<i32>::new();

    // ... populate v ...

    if let Some(x) = v.get(0) {

        // use x

    } else {

        // 处理错误

    }

}

```

当错误处理涉及某种控制流操作，如 `break` 或 `return` 时，`let else` 语法甚至更简洁。

与正常的 `let` 语句类似，`let else` 语句只能在期望语句的地方使用。`let else` 语句还意味着否则的情况不是正常情况，并且不会发生进一步的（正常）处理。

```rs

fn main() {

    let mut v = Vec::<i32>::new();

    // ... 填充 v ...

    let Some(x) = v.get(0) else {

        // 处理错误

        return;

    };

    // 使用 x

}

```

`Result` 和 `Option` 也有一些用于处理错误的辅助方法。这些方法类似于 C++ 中 `std::expected` 的方法。

```rs
#include <expected>
#include <string>

int main() {
  std::expected<int, std::string> res(42);
  auto x(res.transform([](int n) { return n * 2; }));
} 
```

```rs

fn main() {

    let res: Result<i32, String> = Ok(42);

    let x = res.map(|n| n * 2);

}

```

这些辅助方法和其它方法在 `Option` ([`Option`](https://doc.rust-lang.org/std/option/enum.Option.html#implementations)) 和 `Result` ([`Result`](https://doc.rust-lang.org/std/result/enum.Result.html#implementations)) 的文档中有详细描述。

## 借用结果

在上述示例中，成功的结果是从向量中借用的。通常需要将结果克隆或复制到拥有副本中，并且希望在不需要匹配和重建值的情况下这样做。`Result` 和 `Option` 有助于这些目的的辅助方法。

```rs

fn main() {

    let mut v = Vec::<i32>::new();

    v.push(42);

    let x: Option<&i32> = v.get(0);

    let y: Option<i32> = v.get(0).copied();

    let mut w = Vec::<String>::new();

    w.push("hello".to_string());

    let s: Option<&String> = w.get(0);

    let r: Option<String> = w.get(0).cloned();

}

```

## 传播错误

在 C++ 中，异常会自动传播。在 Rust 中，由 `Result` 或 `Option` 指示的错误必须显式传播。`?` 操作符是这种便利的表示。还有几个用于操作 `Result` 和 `Option` 的方法，它们具有类似传播错误的效果。

```rs
#include <cstddef>
#include <vector>

int accessValue(std::vector<std::size_t> indices,
                 std::vector<int> values,
                 std::size_t i) {
  // vector::at throws
  size_t idx(indices.at(i));
  // vector::at throws
  return values.at(idx);
} 
```

```rs

#![allow(unused)] fn main() { fn access_value(

    indices: Vec<usize>,

    values: Vec<i32>,

    i: usize,

) -> Option<i32> {

    // * 解引用 &i32 以复制它

    // ? 传播 None

    let idx = *indices.get(i)?;

    // 直接返回 Option

    values.get(idx).copied()

}

}

```

上述 Rust 示例等价于以下示例，它没有使用 `?` 操作符。使用 `?` 的版本更符合习惯用法。

```rs

#![allow(unused)] fn main() { fn access_value(

    indices: Vec<usize>,

    values: Vec<i32>,

    i: usize,

) -> Option<i32> {

    // 通过 & 匹配会复制 i32

    let Some(&idx) = indices.get(i) else {

        return None;

    };

    // 仍然直接返回 Option

    values.get(idx).copied()

}

}

```

以下示例也是等效的。它不是惯用的（在这里使用 `?` 更易读），但它确实演示了一个辅助方法。`Option::and_then` 与 C++23 中的 `std::optional::and_then` 类似([`std::optional::and_then` in C++23](https://en.cppreference.com/w/cpp/utility/optional/and_then)).

```rs

#![allow(unused)] fn main() { fn access_value(

    indices: Vec<usize>,

    values: Vec<i32>,

    i: usize,

) -> Option<i32> {

    // 通过 & 匹配会复制 i32

    indices

        .get(i)

        .and_then(|idx| values.get(*idx))

        .copied()

}

}

```

这些辅助方法和其它方法在 `Option` 和 `Result` 的文档中进行了详细描述 [Option](https://doc.rust-lang.org/std/option/enum.Option.html#implementations) 和 [Result](https://doc.rust-lang.org/std/result/enum.Result.html#implementations)。

## `main` 中的未捕获异常

在 C++ 中，当未捕获异常时，它将以非零退出代码和错误消息终止程序。要使用 Rust 中的 `Result` 实现类似的结果，可以将 `main` 的返回类型指定为 `Result`。

```rs
#include <stdexcept>

int main() {
  throw std::runtime_error("oops");
} 
```

```rs
fn main() -> Result<(), &'static str> {
    Err("oops")
}
```

在 Rust 中，`main` 函数可以返回任何实现了 `Termination` 特质的类型，例如 `()`, `Result` 和 `ExitCode`。例如，`main` 可以返回一个错误类型实现了 `Debug` 特质的 `Result`：

```rs

#[derive(Debug)]

struct InterestingError {

    message: &'static str,

    other_interesting_value: i32,

}

fn main() -> Result<(), InterestingError> {

    Err(InterestingError {

        message: "oops",

        other_interesting_value: 9001,

    })

}

```

运行此程序将输出 `Error: InterestingError { message: "oops", other_interesting_value: 9001 }` 并带有退出代码 `1`。

## 使用 `Result` 强制错误处理的限制

当与通过可变引用传递预分配缓冲区的 API 一起使用时，返回 `Result` 或 `Option` 并不提供通常的好处。这是因为缓冲区可以在 `Result` 或 `Option` 之外访问，因此编译器无法强制处理错误情况。

例如，在以下示例中，`read_line` 的结果可以被忽略，导致程序中出现逻辑错误。然而，由于需要初始化缓冲区，它不会导致内存安全违规或未定义的行为。

```rs

fn main() {

    let mut buffer = String::with_capacity(1024);

    std::io::stdin().read_line(&mut buffer);

    // 使用缓冲区

}

```

由于 `Result` 上有 `#[must_use]` 属性，Rust 将在这种情况下产生警告。

```rs
warning: unused `Result` that must be used
 --> example.rs:3:5
  |
3 |     std::io::stdin().read_line(&mut buffer);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: this `Result` may be an `Err` variant, which should be handled
  = note: `#[warn(unused_must_use)]` on by default
help: use `let _ = ...` to ignore the resulting value
  |
3 |     let _ = std::io::stdin().read_line(&mut buffer);
  |     +++++++ 
```

`Option` 没有包含 `#[must_use]` 属性，因此必须处理返回 `Option` 的函数（由于 `None` 情况表示错误），应该使用 `#[must_use]` 属性进行注释。例如，切片的 `get` 方法返回 `Option` 并被 [标记为 `#[must_use]`](https://doc.rust-lang.org/src/core/slice/mod.rs.html#592-595)。

## 设计和实现错误类型

与 C++ 相比，在 Rust 中处理错误的一个挑战是，由于 Rust 中的错误传播是显式的，因此来自不同子系统的错误值需要组合成一个单一类型，以便进一步向上传播到堆栈。使用 C++ 异常则不需要特殊努力。

以下示例展示了如何手动实现此类错误类型。后续示例将展示如何使用 [thiserror](https://docs.rs/thiserror/latest/thiserror/) 和 [anyhow](https://docs.rs/anyhow/latest/anyhow/) crate 来减少实现的冗长性。

```rs
#include <exception>

struct ErrorA : public std::exception {
  const char *msg = "ErrorA was produced";
  const char *what() const noexcept override {
    return msg;
  }
};

void mightThrowA() {}

struct ErrorB : public std::exception {
  const char *msg = "ErrorA was produced";
  const char *what() const noexcept override {
    return msg;
  }
};

void mightThrowB() {}

void process() {
  mightThrowA();
  mightThrowB();
} 
```

```rs

#![allow(unused)] fn main() { use std::error::Error;

use std::fmt::Display;

use std::fmt::Formatter;

#[derive(Debug)]

struct ErrorA;

impl Display for ErrorA {

    fn fmt(

        &self,

        fmt: &mut Formatter<'_>,

    ) -> Result<(), std::fmt::Error> {

        write!(fmt, "ErrorA produced")

    }

}

impl Error for ErrorA {}

fn might_throw_A() -> Result<(), ErrorA> {

    Ok(())

}

#[derive(Debug)]

struct ErrorB;

impl Display for ErrorB {

    fn fmt(

        &self,

        fmt: &mut Formatter<'_>,

    ) -> Result<(), std::fmt::Error> {

        write!(fmt, "ErrorB produced")

    }

}

impl Error for ErrorB {}

fn might_throw_B() -> Result<(), ErrorB> {

    Ok(())

}

// 这个额外的结构是必要的，用于组合错误

#[derive(Debug)]

enum ErrorAOrB {

    ErrorA(ErrorA),

    ErrorB(ErrorB),

}

impl Display for ErrorAOrB {

    fn fmt(

        &self,

        fmt: &mut Formatter<'_>,

    ) -> Result<(), std::fmt::Error> {

        match self {

            Self::ErrorA(err) => err.fmt(fmt),

            Self::ErrorB(err) => err.fmt(fmt),

        }

    }

}

impl Error for ErrorAOrB {}

impl From<ErrorA> for ErrorAOrB {

    fn from(err: ErrorA) -> Self {

        Self::ErrorA(err)

    }

}

impl From<ErrorB> for ErrorAOrB {

    fn from(err: ErrorB) -> Self {

        Self::ErrorB(err)

    }

}

fn process() -> Result<(), ErrorAOrB> {

    // ? 操作符使用 From 实例

    might_throw_A()?;

    might_throw_B()?;

    Ok(())

}

}

```

以下示例使用 [thiserror](https://docs.rs/thiserror/latest/thiserror/) crate 来实现与上述示例相同的功能。用于比较的 C++ 版本与上一个示例相同。

```rs
#include <exception>

struct ErrorA : public std::exception {
  const char *msg = "ErrorA was produced";
  const char *what() const noexcept override {
    return msg;
  }
};

void mightThrowA() {}

struct ErrorB : public std::exception {
  const char *msg = "ErrorA was produced";
  const char *what() const noexcept override {
    return msg;
  }
};

void mightThrowB() {}

void process() {
  mightThrowA();
  mightThrowB();
} 
```

```rs
use thiserror::Error;

#[derive(Debug, Error)]
#[error("ErrorA was produced")]
struct ErrorA;

fn might_throw_A() -> Result<(), ErrorA> {
    Ok(())
}

#[derive(Debug, Error)]
#[error("ErrorB was produced")]
struct ErrorB;

fn might_throw_B() -> Result<(), ErrorB> {
    Ok(())
}

#[derive(Debug, Error)]
enum ErrorAOrB {
    #[error("error from source A")]
    ErrorA(#[from] ErrorA),
    #[error("error from source B")]
    ErrorB(#[from] ErrorB),
}

fn process() -> Result<(), ErrorAOrB> {
    might_throw_A()?;
    might_throw_B()?;
    Ok(())
}
```

## 应用程序的错误类型

当实现一个应用程序（而不是库）时，通常情况下，具体的错误类型并不像轻松传播错误的能力那样重要。对于这些情况，[anyhow](https://crates.io/crates/anyhow) crate 提供了将错误组合成单个错误类型的机制，以及生成一次性错误的 capability。由于与 anyhow 一起使用的错误类型仍然需要实现 `std::error::Error` trait，anyhow 通常与 thiserror 一起使用。

根据错误的类型进行区分，就像在 C++ 中使用 `catch` 一样，可以使用其中的一个 `【downcast】方法](https://docs.rs/anyhow/latest/anyhow/struct.Error.html#method.downcast)`。

```rs
#include <exception>

struct ErrorA : public std::exception {
  const char *msg = "ErrorA was produced";
  const char *what() const noexcept override {
    return msg;
  }
};

void mightThrowA() {}

struct ErrorB : public std::exception {
  const char *msg = "ErrorA was produced";
  const char *what() const noexcept override {
    return msg;
  }
};

void mightThrowB() {}

void process() {
  mightThrowA();
  mightThrowB();
}

int main() {
  try {
    process();
  } catch (ErrorA &err) {
    // handle ErrorA
  } catch (ErrorB &err) {
    // handle ErrorB
  }
} 
```

```rs
use thiserror::Error;

#[derive(Debug, Error)]
#[error("ErrorA was produced")]
struct ErrorA;

fn might_throw_A() -> Result<(), ErrorA> {
    Ok(())
}

#[derive(Debug, Error)]
#[error("ErrorB was produced")]
struct ErrorB;

fn might_throw_B() -> Result<(), ErrorB> {
    Ok(())
}

fn process() -> anyhow::Result<()> {
    might_throw_A()?;
    might_throw_B()?;
    Ok(())
}

fn main() {
    if let Err(err) = process() {
        if let Some(errA) =
            err.downcast_ref::<ErrorA>()
        {
            // handle ErrorA
        } else if let Some(errB) =
            err.downcast_ref::<ErrorB>()
        {
            // handle ErrorB
        }
    }
}
```

## 回溯

可以通过定义一个类型为 `Backtrace` 的字段来手动将回溯信息包含在错误中。[`Backtrace::capture` 方法](https://doc.rust-lang.org/std/backtrace/struct.Backtrace.html#method.capture) 可以用来捕获回溯。模块文档[描述了启用回溯所需的配置](https://doc.rust-lang.org/std/backtrace/index.html)。

两者 [thiserror](https://docs.rs/thiserror/latest/thiserror/) 和 [anyhow](https://docs.rs/anyhow/latest/anyhow/) 都支持方便地将回溯信息添加到错误中。关于包含回溯的说明可以在每个 crate 的主文档页面上找到。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Expected%20errors)
