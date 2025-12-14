# Tests

> 原文：[`cel.cs.brown.edu/crp/etc/tests.html`](https://cel.cs.brown.edu/crp/etc/tests.html)

本章通过将 Rust 中的测试与 C++ 的 [Boost.Test](https://github.com/boostorg/test) 进行比较，提供了一个小的测试示例。关于 Rust 中测试的更详细指南可以作为 [Rust Book](https://doc.rust-lang.org/book/ch11-00-testing.html) 的一部分找到，包括有关 [文档测试](https://doc.rust-lang.org/stable/book/ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests) 的信息。关于 [rustc](https://doc.rust-lang.org/rustc/tests/index.html) 和 [Cargo](https://doc.rust-lang.org/cargo/commands/cargo-test.html) 内置测试支持的详细信息可以在它们各自的手册中找到。

## 单元测试

当使用 C++ 测试框架，如 Boost.Test 时，测试被定义为与框架注册的函数。测试是针对框架提供的测试驱动程序编译的。

在 Rust 中，测试的处理方式类似。与 C++ 不同，测试注册机制和测试驱动程序都是由 Rust 本身提供的。

此外，测试的组织方式也不同。在 C++ 中，单元测试是在被测试的单元之外单独定义的文件中定义的。在 Rust 中，它们通常在同一个文件中定义，通常在 `test` 子模块中，模块的包含由 `test` 功能标志通过 `#[cfg(test)]` 注解控制。

下面的示例定义了一个小的类及其测试。在 C++ 中，这涉及到创建三个单独的文件：一个接口的头文件、接口的实现以及测试驱动程序。在 Rust 中，这一切都在一个文件中完成。

```rs
// counter.h
#ifndef COUNTER_H
#define COUNTER_H

class Counter {
  unsigned int count;

public:
  Counter();
  unsigned int get();
  void increment();
};

#endif

// counter.cc
#include "counter.h"
Counter::Counter() : count(0) {}

unsigned int Counter::get() {
  return count;
}

void Counter::increment() {
  ++count;
}

// test_main.cc
#define BOOST_TEST_MODULE my_tests
#include <boost/test/included/unit_test.hpp>
#include "counter.h"

BOOST_AUTO_TEST_CASE(test_counter_initialize) {
  Counter c;
  BOOST_TEST(c.get() == 0);
}

BOOST_AUTO_TEST_CASE(test_counter_increment) {
  Counter c;
  c.increment();
  BOOST_TEST(c.get() == 1);
} 
```

```rs

#![allow(unused)] fn main() { // counter.rs

pub struct Counter(u32);

impl Counter {

    pub fn new() -> Counter {

        Counter(0)

    }

    pub fn get(&self) -> u32 {

        self.0

    }

    pub fn increment(&mut self) {

        self.0 += 1;

    }

}

#[cfg(test)]

mod test {

    use super::*;

    #[test]

    fn counter_initialize() {

        let c = Counter::new();

        assert_eq!(0, c.get());

    }

    #[test]

    fn counter_increment() {

        let mut c = Counter::new();

        c.increment();

        assert_eq!(1, c.get());

    }

}

}

```

在被测试的模块内定义单元测试的好处是使被测试模块的内部结构对测试代码可见。这使得在不暴露给程序的其他部分的情况下（如在 C++ 中通过在头文件中包含声明并将测试固定点作为 `friend` 声明）对内部组件进行单元测试成为可能。

在 C++ 中运行测试涉及链接到 `boost_unit_test_framework`。在 Rust 中，可以使用 `cargo test` 运行测试。如果不使用 Cargo，可以通过将标志 `--test` 传递给 rustc 来编译测试，并运行生成的可执行文件。

## 集成测试

Cargo 支持的集成测试仍然是在模块外部和包外部定义的，纯粹基于暴露的 API。

请参阅[Rust 书籍](https://doc.rust-lang.org/book/ch11-03-test-organization.html#integration-tests)以了解如何为 Rust 程序组织集成测试的详细信息。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Tests)。
