# Constructors

> 原文：[`cel.cs.brown.edu/crp/idioms/constructors.html`](https://cel.cs.brown.edu/crp/idioms/constructors.html)

在 C++ 中，构造函数初始化对象。当构造函数执行时，对象已分配存储空间，构造函数只执行初始化。

Rust 与 C++ 不同，没有相同的构造函数。在 Rust 中，创建对象有唯一的基本方式，即一次性初始化其所有成员。Rust 中的“构造函数”或“构造函数方法”更像是工厂：与类型相关联的静态方法（即，没有 `self` 参数的方法），它返回该类型的值。

```rs
#include <thread> unsigned int cpu_count() {
 return std::thread::hardware_concurrency(); }   class ThreadPool {
  unsigned int num_threads;

public:
  ThreadPool() : num_threads(cpu_count()) {}
  ThreadPool(unsigned int nt) : num_threads(nt) {}
};

int main() {
  ThreadPool p1;
  ThreadPool p2(4);
} 
```

```rs

fn cpu_count() -> usize {

std::thread::available_parallelism().unwrap().get() }   struct ThreadPool {

num_threads: usize

}

impl ThreadPool {

    fn new() -> Self {

        Self { num_threads: cpu_count() }

    }

    fn with_threads(nt: usize) -> Self {

        Self { num_threads: nt }

    }

}

fn main() {

    let p1 = ThreadPool::new();

    let p2 = ThreadPool::with_threads(4);

}

```

在 Rust 中，通常一个类型的首选构造函数被命名为 `new`，尤其是当它不接受任何参数时。（参见关于默认构造函数的章节。）基于值的某些特定属性的构造函数通常命名为 `with_<something>`，例如，`ThreadPool::with_threads`。请参阅[Rust 命名指南](https://rust-lang.github.io/api-guidelines/naming.html)，了解 Rust 中构造函数方法的命名约定。

如果要初始化的字段是可见的，有合理的默认值，并且该值不管理资源，那么使用记录更新语法根据某些默认值初始化值也是常见的。

```rs

struct Point {

    x: i32,

    y: i32,

    z: i32,

}

impl Point {

    const fn zero() -> Self {

        Self { x: 0, y: 0, z: 0 }

    }

}

fn main() {

    let x_unit = Point {

        x: 1,

        ..Point::zero()

    };

}

```

尽管名为“记录更新语法”，但它并不修改记录，而是基于另一个值创建一个新的值，为了这样做而获取其所有权。

## 存储分配与初始化

在 Rust 中，结构体或枚举值的实际构造发生在结构体构造语法（例如，`ThreadPool { ... }`）的位置，在评估字段的表达式（例如，`cpu_count()`）之后。

这种差异的一个重要影响是，在 Rust 中，在调用构造方法（如 `ThreadPool::with_threads`）时不会为结构体分配存储，实际上是在计算结构体字段值之后（从语言语义的角度来看——优化器可能仍然会避免复制）。因此，在 Rust 中没有直接的方法来翻译类似在构造时存储指向自身指针的类（在 Rust 中，这需要像 `Pin` 和 `MaybeUninit` 这样的工具）的模式。

## 可失败构造函数

在 C++ 中，构造函数可以通过抛出异常来指示失败。在 Rust 中，因为构造函数是普通的静态方法，所以可失败构造函数可以返回 `Result`（类似于 `std::expected`）或 `Option`（类似于 `std::optional`）。^(1)

```rs
#include <iostream>
#include <stdexcept>

class ThreadPool {
  unsigned int num_threads;

public:
  ThreadPool(unsigned int nt) : num_threads(nt) {
    if (num_threads == 0) {
      throw std::domain_error(
          "Cannot have zero threads");
    }
  }
};

int main() {
  try {
    ThreadPool p(0);
    // use p here
  } catch (const std::domain_error &e) {
    std::cout << e.what() << std::endl;
  }
} 
```

```rs

struct ThreadPool {

    num_threads: usize,

}

#[derive(Debug)]

enum ThreadPoolError {

    ZeroThreads,

}

impl ThreadPool {

    fn with_threads(

        nt: usize,

    ) -> Result<Self, ThreadPoolError> {

        如果 nt 等于 0 {

            Err(ThreadPoolError::ZeroThreads)

        } else {

            Ok(Self { num_threads: nt })

        }

    }

}

fn main() {

    match ThreadPool::with_threads(0) {

        Err(err) => println!("{:?}", err),

        Ok(p) => {

            // 使用 p 这里

        }

    }

}

```

有关 C++ 异常和异常处理如何转换为 Rust 的更多信息，请参阅异常章节。

* * *

1.  这里的一个替代方法是将 `NonZero<usize>` 作为类型，这样在最初就不可能发生错误情况。↩

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Constructors)
