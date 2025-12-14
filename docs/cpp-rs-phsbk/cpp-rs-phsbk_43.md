# Placement new

> 原文：[`cel.cs.brown.edu/crp/idioms/placement_new.html`](https://cel.cs.brown.edu/crp/idioms/placement_new.html)

本章中关于 Rust 的一些陈述取决于编译器如何优化各种程序的具体细节。除非另有说明，此处展示的结果是基于使用[2024 语言版本](https://doc.rust-lang.org/edition-guide/introduction.html)的 rustc 1.87。

C++中 placement new 的主要目的是

+   存储分配与初始化分离的情况，例如在`std::vector`或内存池的实现中。

+   需要将结构放置在特定内存位置的情况，例如，与内存映射寄存器一起工作，以及

+   出于性能原因的存储重用。

你也可能是因为寻找如何在 Rust 中直接在堆上构造大值的方法而来到这个页面。

有一个[开放提案](https://github.com/rust-lang/rfcs/pull/2884)在 Rust 中添加与 placement new 类似的功能，但该功能的架构仍在讨论中。同时，对于 placement new 的许多用例，在安全 Rust 中存在替代方案，或者使用不安全 Rust 的方法可以完成所需的行为。

## 自定义分配器和自定义容器

使用 placement new 的第一个原因不常见，因为大多数用例已经被使用带有自定义分配器的 STL 容器所覆盖。同样，Rust 的标准库也可以与自定义分配器一起使用。然而，在 Rust 中，自定义分配器的 API 仍然[不稳定](https://github.com/rust-lang/rust/issues/32838)，因此它们只能在使用带有[特性标志](https://doc.rust-lang.org/unstable-book/library-features/allocator-api.html)的 nightly 编译器时才可用。Rust Book 提供了[如何安装 nightly 工具链的说明](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#unstable-features)，而 Rust Unstable Book 提供了[如何使用不稳定特性的说明](https://doc.rust-lang.org/unstable-book/)。

对于稳定的 Rust，存在许多覆盖分配器用例的库。例如，[bumpalo](https://docs.rs/bumpalo/latest/bumpalo/)提供了一个对 bump 分配场的安全接口，一个[使用该场的向量类型](https://docs.rs/bumpalo/latest/bumpalo/collections/vec/struct.Vec.html)，以及其他使用该场的实用类型。

对于实现涉及单独分配和初始化内存的自定义集合类型，Rustonomicon 中关于[实现`Vec`](https://doc.rust-lang.org/nomicon/vec/vec.html)的章节是一个有用的资源。

## 内存映射寄存器和嵌入式开发

如果你正在使用 Rust 进行嵌入式开发，你可能还想额外阅读[嵌入式 Rust 书籍](https://docs.rust-embedded.org/book/)。关于[外设](https://docs.rust-embedded.org/book/peripherals/index.html)的章节讨论了如何与位于内存中特定地址的结构一起工作。

嵌入式 Rust 书籍还包括[一个章节，为使用 Rust 进行嵌入式开发的嵌入式 C 程序员提供建议](https://docs.rust-embedded.org/book/c-tips/index.html)。

## 性能和存储重用

在 C++中使用 placement new 来重用存储的这种用法通常可以用 Rust 中的简单赋值来替代。因为 Rust 中的赋值总是移动操作，并且 Rust 中的移动不会留下需要销毁的对象，优化器通常会为这种用例生成与 placement new 类似的代码。在某些情况下，这也取决于 RVO 或 NRVO 优化。虽然这些优化不是保证的，但它们对于常见的编码模式来说足够可靠，尤其是在与[基准测试](https://bheisler.github.io/criterion.rs/book/index.html)性能敏感的代码结合使用时，可以确认所需的优化已执行。此外，可以使用像[cargo-show-asm](https://github.com/pacak/cargo-show-asm)这样的工具来检查特定函数生成的汇编代码。

以下示例的 Rust 版本依赖于[优化](https://godbolt.org/#z:OYLghAFBqd5TKALEBjA9gEwKYFFMCWALugE4A0BIEAZgQDbYB2AhgLbYgDkAjF%2BTXRMiAZVQtGIHgBYBQogFUAztgAKAD24AGfgCsp5eiyahSAVyVFyKxqiIEh1ZpgDC6embZMpATnLOAGQImbAA5TwAjbFIQADZyAAd0JWIHJjcPL19E5NShIJDwtiiY%2BJtsOzSRIhZSIgzPbx4/csqhatqiArDI6LjrGrqGrOaBzu6ikriASmt0M1JUTi4AUgAmAGYVgFYAIRxSAgA3bAgAEWwaFjN6Immds5WtAEFLczsAagDa4GwAdQImF%2BRA%2BKwA7Lsns8PjCPoCQB8LAQAF7YchQ8GPF4YzY7XZmJhKFg0U5MdAAfTYxmAjHu2yxzxoTA%2BmApAHcyABrCBshHrWJsMwg76kX4AoHYO6giFQ2EfQSkOFw5laAB0qp4Wi00shLzlcoAVGzQRszl8fv9AcCdbL9Xb4XD0Xq7dLHhtddCXQB6L2IlQfNm2u2WTAgEBIYJEMMRIyoTnkiLodQQfls%2B7uoOujFghlQpkfKnBCDTG3OmGMEGCkFKVCkFhEVBIE1mkViq2SsM4K43IjFlYZsss9lclNrAVCj41usNpDpj2YqFcWb0bjbfjeLg6cjobgAJQs1fmi2woM2fHIUc3S9mnJA2zWquksTB21iGwAHD5PzwZBtDNxpH4Ng7y0cgNy3HcuH4JQQFAy8dFmOBYBQDAcHwYgyEoag6EYVgOG4c9BGEMQJE4GQ5GEZQ1E0K9yH0NZDGpNB1khNZdmsbBbHsRwIGcYZvDWHh/CYTAJl6GJZCSFJuPSdxGhAQScmktIxOKPpZFaGSOiGOSskUzSqkGLpgh6NSJLGHTMgEoTLHGEzJnU2YlCPJZuHWDZglQDwcBNFxUBDLt%2B1wHEPKYLyzB8/sXBCQMNmC7EXjeMxPlbS0JRBcEPRhEMwxSVFyRBQF%2B3nHNioxF4iGwNgEiMSrfKIABPBJmHYE8ABUgqhbB1Eq0hmSOdBAQ%2BGMWDjXYkwgNrT1idQ53K54BqG1k/hHRbMA%2BA0p3rRsS0y20FQ%2BCAcpAPLsAKpV%2BzNLRioujYXA%2BTVrvdOEWJY3aZUHa4SA2tkIBiw6tpnEtUvFYFMtVIbLpenM5sHH0/RPQNBxGsaJrTMrBzZABaIKAD8QfbXtYc9BdsVKhLnkjAsWCLd6so%2BCQCGAVglAgRnmfQGgIAJ9LphLRtakzGEODYMgGrxU7Oe5i1QclOk3XptafubAsqrFjHPWWkd0YHZ5SeeJd/y4NcwO0fhIJcV62MnFyT3ctZ%2BHg69yCQbAWAOahlwAoCQNN2jIOg2CLzNvnyFve9H2fV8Py/Hwf2kP8Vy4DZ1zN7duEdkPyCQhB4AgFD0GqhhoiwiAMCLxgYlIHh3zBUCcN6mCIAiNOImCWoGoI/g29YUgGoAeQiXQKkvc9y44YR%2B6YehO9onAIjMYAXAkegYN4fgcCpExJDnghSBH45sDXrduoqIVli3SNOLT%2BgCAiOs%2B7cHA06IQ5gPX8gTlIRMVAuLeaWCKAK8swaBGGAEoAAagQbAbJ%2B7NQ3IReQJFJDkSIooFQGg076CEkYEwIBzCWEMHfGCkBZjoASDJNeWMsb3XIUQLGjATj0Eug7bcX9Dg4BIcWDiXE0hOBEvxAwgR7LiQMFJPIskrJiNyDJVSUwbKcQPu0IygiFG8OUXZQooibIqN0k0CyxktFmSkE5W2JijYm3AubbgHwCENgeqqWuqptQQHQiQRU7keDTEzsA7OKB5hEASEKUu5cEjF1IKEVq3A7GoAcU4k%2BhB3HwiEmg5BZFZBoKopg2i2DyBsjrAkLuXtjap39twfuQogkgk5rYg8sSeCOLBM4w6bgK7RFPBsLxPiEI3hANIJp2wNhrATrETUmp3zvm2O%2BI2gFyDAW2KBKx6coLWCDk7RCiB85oELmEyuISdnhJAMAHg2wUkMEbtQFutEe4dy7uQG5fdB7DzsHc8ezAiBTxnmneei9l70FXncze1Id5bkIPvSoJxj78FPqgc%2Bdyr5Jy3Lfe%2BHcn4X0dm/O5X8f7YD/sCwBWdQEsHAVAmBcDmB3NSeIFBGT5BZJolueijE8GoCtuxZFXCyEULSFQkMl1WVsRYpsD4WN%2B6sPQOwwER94BOUUW0bwvEBF6KESJORfQhLiJkqopSEi1UxDUUopg2l6jKoNfK41eqDC2UsvJHRmjTLyNMQsVyaximWLTpBD46h3yxCxrEaQHxgCoHqdsDUh03FkA6a67phstkBKqfstpESolcG9b6/1gbg0PVDeebAiSyDwgYlS0iUhaWUQwQyvQCk8kFKKRY0pEFymVInDUtNfqA1BpDWGiArTdntPtt44OwDZiu3dn0bhSdZnzMWR6jOqy4Ih16f01UgzhkbFGVocZkzplJxTn7RtKz1nFNYUsgOQ6emf2iCkRw0ggA%3D%3D)来实现所需的行为。

```rs
#include <cstddef>
#include <new>

struct LargeWidget {
  std::size_t id;
};

template <typename T>
extern void blackBox(T &x);

void doWork(void *scratch) {
  for (std::size_t i = 0; i < 100; i++) {
    auto *w(new (scratch) LargeWidget{.id = i});
    // use w
    blackBox(w);
    w->~LargeWidget();
  }
}

int main() {
  alignas(alignof(LargeWidget)) char
      memory[sizeof(LargeWidget)];
  void *w = memory;
  doWork(w);
} 
```

```rs

```

#[derive(Default)]

struct LargeWidget {

    id: usize,

}

fn do_work(w: &mut LargeWidget) {

    for i in 0..100 {

        *w = LargeWidget { id: i };

        // use w

        std::hint::black_box(&w);

    }

}

fn main() {

    let mut scratch = LargeWidget::default();

    do_work(&mut scratch);

}

```rs

```

为 `LargeWidget` 添加 `Drop` 实现会导致在每次循环迭代时调用析构函数，但这会使生成的汇编代码难以阅读，因此已从示例中省略。

## 在堆上构造大值

`new` 在 C++ 构造对象直接在动态存储中，而 placement `new` 直接在提供的位置构造它们。在 Rust 中，`Box::new` 是一个普通函数，因此值是在栈上构造的，然后移动到堆（或由自定义分配器提供的存储）。

虽然值在栈上的初始构造有时可以被优化掉，但为了保证不使用栈来存储大值，需要使用不安全的 Rust 和 `MaybeUninit`。此外，用于在堆上初始化值的机制并不能保证值不会首先在栈上创建然后移动到堆上。相反，它们只是使逐步初始化结构（无论是按字段还是按元素）成为可能，这样整个结构就不必一次性全部位于栈上。然而，相同的优化也适用，因此可能避免额外的复制。 

```rs
#include <iostream>
#include <memory>

int main() {
  constexpr unsigned int SIZE = 8000000;
  std::unique_ptr b = std::make_unique<
      std::array<unsigned int, SIZE>>();
  for (std::size_t i; i < SIZE; ++i) {
    (*b)[i] = 42;
  }

  // use b so that it isn't optimized away
  for (std::size_t i; i < SIZE; ++i) {
    std::cout << (*b)[i] << std::endl;
  }
} 
```

```rs

```

fn main() {

    const SIZE: usize = 8_000_000;

    // 此处的优化避免了溢出

    // 使用 opt-level=2 的栈

    let mut b = Box::new([0; SIZE]);

    for i in 0..SIZE {

        b[i] = 42;

    }

    // 使用 b 以确保它不会被优化掉

    std::hint::black_box(&b);

}

```rs

```

在另一方面，直接将数组定义为 `[42; SIZE]` 会导致值首先在栈上构造，这会在运行时产生错误。

```rs

```

fn main() {

    const SIZE: usize = 8_000_000;

    let b = Box::new([42; SIZE]);

    // 使用 b 以确保它不会被优化掉

    std::hint::black_box(&b);

}

```rs

```

```rs
thread 'main' has overflowed its stack
fatal runtime error: stack overflow
Aborted (core dumped) 
```

虽然直接在堆上构造值无法强制执行，但可以通过使用不安全的 Rust 来逐步构造值，从而避免栈溢出。这种技术依赖于 `MaybeUninit` 和 `addr_of_mut!`。

```rs

```

fn main() {

    const SIZE: usize = 8_000_000;

    let mut b = Box::<[i32; SIZE]>::new_uninit();

    let bptr = b.as_mut_ptr();

    for i in 0..SIZE {

        unsafe {

            std::ptr::addr_of_mut!(((*bptr)[i])).write(42);

        }

    }

    let b2 = unsafe { b.assume_init() };

    for i in 0..SIZE {

        println!("{}", b2[i]);

    }

}

```rs

```

根据需要，这种特定用法可以被泛化。

```rs

```

#![allow(unused)] fn main() { fn init_with<T, const SIZE: usize>(

    f: impl Fn(usize) -> T,

) -> Box<[T; SIZE]> {

    let mut b = Box::<[T; SIZE]>::new_uninit();

    let bptr = b.as_mut_ptr();

    for i in 0..SIZE {

        unsafe {

            std::ptr::addr_of_mut!(((*bptr)[i]))

                .write(f(i));

        }

    }

    unsafe { b.assume_init() }

}

}

```rs

```

注意，处理堆上的大数组时，更符合习惯的方式是将它表示为 boxed slice 或 vector，而不是 boxed array。在这种情况下，使用迭代器定义值可以避免在栈上构造它，并且不需要使用 unsafe Rust。

```rs

```

#![allow(unused)] fn main() { fn init_with<T, const SIZE: usize>(

    f: impl Fn(usize) -> T,

) -> Box<[T]> {

    (0..SIZE).map(f).collect()

}

}

```rs

```

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Placement new)
