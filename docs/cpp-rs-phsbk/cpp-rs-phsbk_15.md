# 模板特化

> 原文：[`cel.cs.brown.edu/crp/idioms/data_modeling/template_specialization.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/template_specialization.html)

C++ 中的模板特化使得模板实体可以为不同的参数有不同的实现。大多数 STL 实现都利用这一点，例如，提供 `[std::vector<bool>` 的空间高效表示](https://en.cppreference.com/w/cpp/container/vector_bool)。

由于模板特化的可能性，当一个 C++ 函数操作 `std::vector` 这样的模板类值时，该函数本质上是在模板类提供的接口的术语下定义的，而不是针对特定实现。

要在 Rust 中实现相同的功能，需要根据它操作的接口特质来定义函数。这使得客户端可以通过使用实现该接口的任何具体类型来选择他们选择的数据表示。

在 Rust 中这样做比在 C++ 中更实用，因为泛型不是一种通用的元编程设施意味着泛型实体可以在本地进行类型检查，这使得它们更容易定义。在 Rust 中比在 C++ 中更常见，因为 Rust 没有实现继承，因此在接口和实现之间存在比 C++ 中更清晰的界限。

以下示例展示了如何实现 Rust 函数，以便客户端可以选择不同的具体表示。对于紧凑的位向量表示，示例使用了来自 [bitvec crate](https://docs.rs/bitvec/latest/bitvec/) 的 `BitVec` 类型。`BitVec` 的目的是提供类似于 `Vec<bool>` 或 `std::vector<bool>` 的 API。

```rs
#include <string>
#include <vector>

template <typename T>
void push_if_even(int n,
                  std::vector<T> &collection,
                  T item) {
  if (n % 2 == 0) {
    collection.push_back(std::move(item));
  }
}

int main() {
  // Operate on the default std::vector
  // implementation
  std::vector<std::string> v{"a", "b"};
  push_if_even(2, v, std::string("c"));

  // Operate on the (likely space-optimized)
  // std::vector implementation
  std::vector<bool> bv{false, true};
  push_if_even(2, bv, false);
} 
```

```rs
// The Extend trait is for types that support
// appending values to the collection.
fn push_if_even<T, I: Extend<T>>(
    n: u32,
    collection: &mut I,
    item: T,
) {
    if n % 2 == 0 {
        collection.extend([item]);
    }
}

use bitvec::prelude::*;

fn main() {
    // Operate on Vec
    let mut v =
        vec!["a".to_string(), "b".to_string()];
    push_if_even(2, &mut v, "c".to_string());

    // Operate on BitVec
    let mut bv = bitvec![0, 1];
    push_if_even(2, &mut bv, 0);
}
```

## 泛型和模板之间的权衡

由于泛型函数只能以特质界限定义的方式与泛型值交互，因此测试泛型实现更容易。特别是，测试泛型实现的代码只需要考虑给定特质可能的行为。

为了进行比较，考虑以下程序。

```rs
template <totally_ordered T>
T max(const T &x, const T &y) {
  return (x > y) ? x : y;
}

template <>
int max(const int &x, const int &y) {
  return (x > y) ? x + 1 : y + 1;
} 
```

```rs

#![allow(unused)] fn main() { fn max<'a, T: Ord>(x: &'a T, y: &'a T) -> &'a T {

    if x > y {

        x

    } else {

        y

    }

}

}

```

在 Rust 程序中，*参数化*意味着（假设是安全的 Rust）仅从类型本身就可以判断，如果函数返回，它必须返回 `x` 或 `y` 中的一个。这是因为特质界限 `Ord` 不提供任何构造类型 `T` 的新值的方法，而引用的使用也不提供函数从早期调用中存储 `x` 或 `y` 以在后续调用中返回的方法。

在 C++ 程序中，使用 `int` 作为模板参数调用 `max` 函数将产生与任何其他参数明显不同的结果，这是因为模板特化使得函数的行为可以根据类型而变化。

代价是，在 Rust 中，特化的实现更难使用，因为它们必须有不同的名称，但它们更容易编写，因为在编写泛型代码的同时可以更有信心地保证其正确性。

## 利基优化

在某些情况下，Rust 编译器将执行优化以实现更有效的表示。这些情况都是那些效率提升不会改变代码可观察行为的情况。

[最常见的案例是使用 `Option` 类型](https://doc.rust-lang.org/std/option/index.html#representation)。当 `Option` 与编译器可以确定存在未使用值的类型一起使用时，其中一个未使用的值将被用来表示 `None` 情况，这样 `Option<T>` 就不需要额外的内存字来指示枚举的区分符。

这种优化应用于引用类型（`&` 和 `&mut`），因为引用不能为空。它也应用于 `NonNull<T>`，它表示指向类型 `T` 值的非空指针，以及 `NonZeroU8` 和其他非零整型。针对引用情况的优化使得 `Option<&T>` 和 `Option<&mut T>` 成为使用 C++ 中的非拥有观察指针的更安全等价物。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Template specialization)
