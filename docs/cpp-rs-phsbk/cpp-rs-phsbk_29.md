# 类型等效

> 原文：[`cel.cs.brown.edu/crp/idioms/type_equivalents.html`](https://cel.cs.brown.edu/crp/idioms/type_equivalents.html)

本文档中列出的类型等效项在 Rust 编程中与 C++ 编程中的类型等效，但并不一定在通过 FFI 与 C 或 C++ 程序交互时等效。对于与 C 或 C++ 互操作有用的类型，请参阅 [Rust `std::ffi` 模块文档](https://doc.rust-lang.org/std/ffi/index.html) 和 [Rustonomicon 中的 FFI 文档](https://doc.rust-lang.org/nomicon/ffi.html)。

## 原始类型

### 整数类型

在 C++ 中，许多整数类型（如 `int` 和 `long`）的宽度是实现的定义。在 Rust 中，整数类型总是指定它们的宽度，类似于 C++ 中的 `<cstdint>` 类型。当不清楚使用哪种整数类型时，[通常默认使用 `i32`，这是 Rust 为整数字面量默认的类型](https://doc.rust-lang.org/book/ch03-02-data-types.html#integer-types)。

| C++ 类型 | Rust 类型 |
| --- | --- |
| `uint8_t` | `u8` |
| `uint16_t` | `u16` |
| `uint32_t` | `u32` |
| `uint64_t` | `u64` |
| `int8_t` | `i8` |
| `int16_t` | `i16` |
| `int32_t` | `i32` |
| `int64_t` | `i64` |
| `size_t` | `usize` |
| `ptrdiff_t` | `isize` |

在 C++ 中，`size_t` 传统上仅用于索引、大小和偏移量。在 Rust 中，`usize` 也是这样，它是指针大小的无符号整数类型。指针大小的有符号整数类型 `isize` 遵循类似的传统。

一些原始的 C++ 和 POSIX 类型（如 `ssize_t` 和 `off_t` 作为返回类型）不映射到 `isize`，因为错误情况是用 `std::io::Result` 表示，而不是用负数作为 哨兵值。其他类型（如 `fpos_t` 或 `off_t` 作为参数类型）则表示为普通的 `u64` 或在 Rust 中有更明确的表示。

| C++ 类型 | Rust 类型 |
| --- | --- |
| POSIX `ssize_t` | [`std::io::Result<u64>`](https://doc.rust-lang.org/std/io/type.Result.html) |
| POSIX `off_t` (作为参数) | `u64` |
| POSIX `off_t`(作为返回值) | [`std::io::Result<u64>`](https://doc.rust-lang.org/std/io/type.Result.html) |
| `fpos_t` | [`std::io::SeekFrom`](https://doc.rust-lang.org/std/io/enum.SeekFrom.html) |

### 浮点类型

与 C++ 中的整数类型一样，浮点类型 `float`、`double` 和 `long double` 的宽度是实现的定义。C++23 引入了保证为特定宽度的 IEEE 754 浮点类型。其中，`float32_t` 和 `float64_t` 对应于通常从 `float` 和 `double` 期望的类型。Rust 的浮点类型与此类似。

| C++ 类型 | Rust 类型 |
| --- | --- |
| `float16_t` |  |
| `float32_t` | `f32` |
| `float64_t` | `f64` |
| `float128_t` |  |

Rust 中与 `float16_t` 和 `float128_t`（`f16` 和 `f128`）相对应的类型在稳定的 Rust 中[尚未提供](https://github.com/rust-lang/rust/issues/116909)。

### 原始内存类型

在 C++ 中，使用 `char`、`unsigned char` 或 `byte` 的指针或数组来表示原始内存。在 Rust 中，使用 `u8` 的数组（`[u8; N]`）、向量（`Vec<u8>`）或切片（`&[u8]`）来完成相同的目标。然而，以这种方式访问另一个 Rust 值的底层内存需要不安全 Rust。有 库 可以创建围绕这种访问的安全包装，用于诸如序列化或与硬件交互等目的。

### 字符和字符串类型

C++ 中的 `char` 或 `wchar_t` 类型具有实现定义的宽度。Rust 没有这些类型的等效类型。在 Rust 中处理字符串编码时，会使用无符号整数类型，而在 C++ 中会使用固定宽度的字符类型。

| C++ 类型 | Rust 类型 |
| --- | --- |
| `char8_t` | `u8` |
| `char16_t` | `u16` |

Rust 的 `char` 类型表示一个 Unicode 标量值。因此，Rust 的 `char` 的大小与 `u32` 相同。对于在 Rust 字符串中处理字符（这些字符串保证是有效的 UTF-8），`char` 类型是合适的。对于表示一个字节，应使用 `u8`。

Rust 标准库包括用于 UTF-8 字符串和字符串切片的类型：`String` 和 `&str`。这两种类型都保证表示的字符串是有效的 UTF-8。Rust 的 `char` 类型适用于表示 `String` 的元素。

因为 `str`（不带引用）是一个切片，它没有大小，因此必须使用类似指针的结构，例如引用或盒式结构。因此，在文档中，字符串切片通常描述为 `&str` 而不是 `str`，尽管它们也可以用作 `Box<str>`、`Rc<str>` 等。

Rust 还包括用于平台特定字符串表示及其字符串切片的类型：`std::ffi::OsString` 和 `&std::ffi::OsStr`。虽然这些字符串使用 OS 特定的表示，但为了使用 Rust FFI，它们必须转换为 `CString`（[`doc.rust-lang.org/std/ffi/struct.CString.html`](https://doc.rust-lang.org/std/ffi/struct.CString.html)）。

与有 `std::u16string` 的 C++ 不同，Rust 没有专门用于 UTF-16 字符串的表示形式。可以使用类似 `Vec<u16>` 的东西，但类型不会保证其内容是有效的 UTF-16 字符串。Rust 也提供了将 `String` 转换为 UTF-16 编码以及从 UTF-16 编码转换回 `String` 的机制（例如 `[String::encode_utf16](https://doc.rust-lang.org/std/string/struct.String.html#method.encode_utf16)` 和 `[String::from_utf16](https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf16)`），以及访问底层 UTF-8 编码的类似机制（[`doc.rust-lang.org/std/string/struct.String.html#method.from_utf8`](https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8)）。

| 目的 | Rust 类型 |
| --- | --- |
| 表示文本 | `String` 和 `&str` |
| 表示字节 | 向量、数组或 `u8` 的切片 |
| 与操作系统交互 | `OsString` 和 `&OsStr` |
| 表示 UTF-8 | `String` |
| 表示 UTF-16 | 使用 一个库 |

### 布尔类型

Rust 中的 `bool` 类型与 C++ 中的 `bool` 类型类似。与 C++ 不同，Rust 对 `bool` 类型的值所使用的尺寸、对齐方式和位模式做出了[保证](https://doc.rust-lang.org/reference/types/boolean.html)。

### `void`

在 C++ 中，`void` 表示函数不返回值。由于 Rust 是面向表达式的，所有函数都返回值。在 `void` 的位置，Rust 使用单位类型 `()`。当函数没有声明返回类型时，`()` 是返回类型。

```rs
#include <iostream>

void process() {
    std::cout
        << "Does something, but returns nothing."
        << std::endl;
} 
```

```rs

#![allow(unused)] fn main() { fn process() {

    println!("Does something but returns nothing.");

}

}

```

由于单位类型只有一个值（也写作 `()`），该类型的值不提供任何信息。这也意味着返回值可以省略，就像上面的示例一样。以下示例使单位类型的用法明确。

```rs

#![allow(unused)] fn main() { fn process() -> () {

    let () = println!("Does something but returns nothing.");

    ()

}

}

```

单位类型和单位值的语法类似于空元组的语法。本质上，类型就是这样。以下示例展示了某些等效类型，尽管没有特殊的语法或语言集成。

```rs

struct Pair<T1, T2>(T1, T2); // 与 (T1, T2) 相同

struct Single<T>(T); // 只包含一个值 (T1) 的元组

struct Unit; // 与 () 相同

// 也可以写成

// struct Unit();

fn main() {

    let pair = Pair(1,2.0);

    let single = Single(1);

    let unit = Unit;

    // 也可以写成

    // let unit = Unit();

}

```

使用单位类型而不是 `void` 允许在需要值的环境中（例如，在 C++ 中会返回 `void` 的函数调用）使用单位类型（如函数调用），这特别有助于定义和使用泛型函数，而不需要像 `std::is_void` 那样为 `void` 类型进行特殊处理。

## 指针

以下表格将 C++ 中的所有权管理类映射到 Rust 中的等效类型。

| 用途 | C++ 类型 | Rust 类型 |
| --- | --- | --- |
| 所有权 | `T` | `T` |
| 单个所有者，动态存储 | `std::unique_ptr<T>` | `Box<T>` |
| 共享所有者，动态存储，不可变，非线程安全 | `std::shared_ptr<T>` | `std::rc::Rc<T>` |
| 共享所有者，动态存储，不可变，线程安全 | `std::shared_ptr<T>` | `std::sync::Arc<T>` |
| 共享所有者，动态存储，可变，非线程安全 | `std::shared_ptr<T>` | [`std::rc::Rc<std::cell::RefCell<T>>`](https://doc.rust-lang.org/book/ch15-05-interior-mutability.html#having-multiple-owners-of-mutable-data-by-combining-rct-and-refcellt) |
| 共享所有者，动态存储，可变，线程安全 | `std::shared_ptr<T>` 与 `std::mutex` | [`std::sync::Arc<std::mutex::Mutex<T>>`](https://doc.rust-lang.org/book/ch16-03-shared-state.html) |
| 常量引用 | `const &T` | `&T` |
| 可变引用 | `&T` | `&mut T` |
| 常量观察者指针 | `const *T` | `&T` |
| 可变观察者指针 | `*T` | `&mut T` |

在 C++ 中，`std::shared_ptr` 的线程安全性比表中显示的要复杂（例如，某些使用可能需要 `std::atomic`）。然而，在安全的 Rust 中，编译器将防止错误使用共享所有者类型。

与 C++ 引用不同，Rust 可以有引用到引用。Rust 的引用更像是观察者指针，而不是像 C++ 引用那样。

### `void*`

Rust 没有与 C++ 中的 `void*` 直接对应的东西。即将到来的关于 `RTTI` 的章节将涵盖一些目标是动态类型的使用案例。Rustonomicon 的 [FFI 章节覆盖了一些使用案例](https://doc.rust-lang.org/nomicon/ffi.html#representing-opaque-structs)，其中目标是与使用 `void*` 的 C 程序进行互操作。

## 容器

C++ 和 Rust 容器都拥有自己的元素。然而，在两者中，元素类型可能是一个非拥有类型，例如 C++ 中的指针或 Rust 中的引用。

| C++ 类型 | Rust 类型 |
| --- | --- |
| `std::vector<T>` | [`Vec<T>`](https://doc.rust-lang.org/std/vec/struct.Vec.html) |
| `std::array<T, N>` | [`[T; N]`](https://doc.rust-lang.org/std/primitive.array.html) |
| `std::list<T>` | [`std::collections::LinkedList<T>`](https://doc.rust-lang.org/std/collections/struct.LinkedList.html) |
| `std::queue<T>` | [`std::collections::VecDeque<T>`](https://doc.rust-lang.org/std/collections/struct.VecDeque.html) |
| `std::deque<T>` | [`std::collections::VecDeque<T>`](https://doc.rust-lang.org/std/collections/struct.VecDeque.html) |
| `std::stack<T>` | [`Vec<T>`](https://doc.rust-lang.org/std/vec/struct.Vec.html) |
| `std::map<K,V>` | [`std::collections::BTreeMap<K,V>`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html) |
| `std::unordered_map<K,V>` | [`std::collections::HashMap<K,V>`](https://doc.rust-lang.org/std/collections/struct.HashMap.html) |
| `std::set<K>` | [`std::collections::BTreeSet<K>`](https://doc.rust-lang.org/std/collections/struct.BTreeSet.html) |
| `std::unordered_set<K>` | [`std::collections::HashSet<K>`](https://doc.rust-lang.org/std/collections/struct.HashSet.html) |
| `std::priority_queue<T>` | [`std::collections::BinaryHeap<T>`](https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html) |
| `std::span<T>` | [`&[T]`](https://doc.rust-lang.org/std/primitive.slice.html) |

对于映射和集合，与容器通过哈希或比较函数参数化相比，类型要求键类型实现 `std::hash::Hash`（无序）或 `std::cmp::Ord`（有序）特性。要使用具有不同哈希或比较函数的容器，必须使用具有不同特性实现的包装类型。

由 STL 提供的许多 C++ 容器类型在 Rust 中没有等效类型。其中许多类型在第三方 库 中有等效可用。

在 C++ 和 Rust 中使用这些类型的一个显著不同之处在于 `Vec<T>` 和数组 `[T; N]` 类型，可以从这些类型中便宜地创建数据的一部分或全部的切片引用 `&[T]` 或 `&mut [T]`。因此，当定义一个不修改向量长度且不需要静态知道数组中元素数量的函数时，将参数作为 `&[T]` 或 `&mut [T]` 而不是所有类型的引用更为习惯。

在 C++ 中，如果可能的话，最好使用开始和结束迭代器或范围而不是 `span`，因为迭代器更通用。同样，在 Rust 中，使用实现 `IntoIter<&T>` 或 `IntoIter<&mut T>` 的泛型类型，而不是 `&[T]` 也是如此。

```rs
#include <iterator>
#include <vector>

template <typename InputIter>
void go(InputIter first, InputIter last) {
  for (auto it = first; it != last; ++it) {
    // ...
  }
}

int main() {
  std::vector<int> v = {1, 2, 3};
  go(v.begin(), v.end());
} 
```

```rs

use std::iter::IntoIterator;

fn go<'a>(iter: impl IntoIterator<Item = &'a mut i32>) {

    for x in iter {

        // ...

    }

}

fn main() {

    let mut v = vec![1, 2, 3];

    go(&mut v);

}

```

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Type equivalents)
