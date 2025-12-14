# 表示错误的错误

> 原文：[`cel.cs.brown.edu/crp/idioms/exceptions/bugs.html`](https://cel.cs.brown.edu/crp/idioms/exceptions/bugs.html)

在 C++ 中，异常有时用于指示由编程错误引起的错误。在许多情况下不会产生异常，而是简单地将 API 的无效使用视为未定义行为。

在 Rust 中，`panic!` 用于这些类型的错误，通常通过 `Result` 或 `Option` 上的 `expect` 或 `unwrap` 方法，或者通过 断言如 `assert!` 来实现。虽然 Rust 中的恐慌可能会回滚堆栈或终止程序，但它永远不会是未定义的行为。

```rs
#include <cstddef>
#include <vector>

int main() {
  std::vector<int> v{1, 2, 3};
  // undefined behavior!
  int x(v[4]);
} 
```

```rs

```

fn main() {

    let v = vec![1,2,3];

    // panics!

    let x = v[4];

}

```rs

```

## 将 `Result` 或 `Option` 转换为 `panic!`

从 `Result` 或 `Option` 转换为恐慌比反过来更容易。因此，许多 Rust 库被编写为返回 `Result` 或 `Option`，并允许调用者通过使用 `unwrap` 或 `expect` 来提取值，如果没有值则引发恐慌，以确定 `None` 结果是否表示错误。

```rs

```

/// 如果数字不能被均匀除尽，则返回 `None`。

fn divide_exact(dividend: i32, divisor: i32) -> Option<i32> {

    let quotient = dividend / divisor;

    if quotient * divisor == dividend {

        Some(quotient)

    } else {

        None

    }

}

// 如果数字不能被 2 整除，则返回 `None`

fn divide_by_two_exact(dividend: i32) -> Option<i32> {

    // divide_exact 返回 None 这里不是错误

    divide_exact(dividend, 2)

}

fn main() {

    let res = divide_exact(10, 3); // 哎呀，一个错误！

    let x = res.unwrap();

    // ...

}

```rs

```

当设计 API 时，如果只提供基于 `Result`（或基于 `Option`）或引发恐慌的接口之一，通常最好提供基于 `Result` 的接口。这样，调用者可以选择省略前置条件检查并处理错误，或者因为前置条件应该得到满足而引发恐慌。

## 断言

在 Rust 中，断言也会引发恐慌。与 C++ 中的 `assert` 不同，Rust 中的 `assert!` 宏家族不能被禁用。因此，当为不安全代码创建安全包装时，它们非常适合断言不变性，除了检查逻辑不变性。

```rs
#include <cassert>
#include <cstddef>

template <typename T>
class Widget {
  T *parts;
  std::size_t partCount;

public:
  // ... constructors ...

  /**
   * @pre n must be smaller than partCount
   */
  T getPart(std::size_t n) {
    // Unlike in Rust, this can be disabled,
    // e.g., with -DNDEBUG.
    assert(n < partCount);
    return *(parts + n);
  }
}; 
```

```rs

```

#![allow(unused)] fn main() { use std::convert::TryFrom;

pub struct Widget<T> {

    }

    part_count: usize,

}

impl<T: Copy> Widget<T> {

    // ... 构造函数方法 ...

    /// 如果 n 大于部分数量，则引发恐慌

    /// 部分。

    pub fn get_part(&self, n: usize) -> T {

        // 安全性：Widget 维护不变性

        // 至少有 part_count 个部分，所以如果 n 是

        // 如果小于部分数量，则我们可以

        // 使用它访问一个部分。

        assert!(

            n < self.part_count,

            "索引 {} 超出部分数量 {}",

            n,

            self.part_count

        );

        let idx = isize::try_from(n).expect(

            "无法将索引转换为偏移量"

        );

        unsafe { self.parts.offset(idx).read() }

    }

}

}

```rs

```

Rust 的 `debug_assert!` 宏更像 C++ 中的 `assert!`，因为它可以通过编译配置选项关闭，因此对于编码在开发和测试期间应该检查但生产中检查成本过高的逻辑不变性非常有用。

### 其他断言宏

Rust 有几个其他便利的断言宏。宏 `[`assert_eq!`](https://doc.rust-lang.org/std/macro.assert_eq.html) 和 `[`assert_ne!`](https://doc.rust-lang.org/std/macro.assert_ne.html) 在断言失败时将使用 `Debug` 特性实现打印它们的参数。

`unreachable!` 宏用于断言在枚举匹配时某些情况预期不可能发生。它本质上与 `panic!` 相同，但具有固定的错误消息，并且更好地传达了意图。

### 静态断言

C++ 也有 `static_assert`，它在模板中使用时保证在编译时评估，除非在模板中使用。当在模板中使用时，如果模板被实例化，它保证在编译时评估。在 Rust 中，可以通过在 const 块或其他某些 [常量上下文](https://doc.rust-lang.org/reference/const_eval.html#const-context) 中调用 `assert!` 来实现相同的效果。便利宏 `assert_eq!` 和 `assert_ne!` （目前）不能在 const 上下文中使用。

以下示例在 Rust 和 C++ 中都无法编译，并显示静态断言的消息。

```rs
#include <cassert>

int main() {
  static_assert(false, "static requirement");
} 
```

```rs

```

fn main() {

    const {

        assert!(false, "静态要求");

    }

}

```rs

```

与 C++ 的 `static_assert` 类似，Rust 中的断言在泛型定义中的 const 块中只有在泛型参数已知时才会评估。以下示例的 C++ 和 Rust 版本只有在 `first` 函数被调用在一个大小小于 1 的数组上时才会编译失败。

```rs
#include <array>
#include <cassert>
#include <cstddef>

template <const std::size_t n>
int &first(std::array<int, n> arr) {
  static_assert(
      n >= 1,
      "array needs to have at last size 1!");
  return arr[0];
} 
```

```rs

```

#![allow(unused)] fn main() { fn first<const N: usize>(arr: [i32; N]) -> i32 {

    const {

        assert!(

            N >= 1,

            "数组至少需要大小为 1！"

        )

    }

    arr[0]

}

}

```rs

```

在 C++ 中，`static_assert` 也可以在命名空间作用域中使用。在 Rust 中实现等效功能需要定义一个未命名的常量。

```rs
static_assert(true,  "top-level assert true");
static_assert(false,  "top-level assert false");

int main() {} 
```

```rs

```

const _: () = assert!(true, "顶级断言为真");

const _: () = assert!(false, "顶级断言为假");

fn main() {}

```rs

```

### 断言和优化器

断言确实会影响 Rust 编译器优化代码的方式（例如，通过使优化器能够消除后续冗余检查），但具体效果并不保证。

## 嵌入式系统中的恐慌

当使用 `#![no_std]` 编程 Rust 嵌入式系统时，没有默认的恐慌处理程序。相反，必须使用 `#[panic_handler]` 注解来指定一个。

《嵌入式 Rust 书籍》[关于处理恐慌的章节](https://docs.rust-embedded.org/book/start/panicking.html)提供了更多关于在`no_std`程序中实现恐慌处理器的详细信息。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Errors indicating bugs)
