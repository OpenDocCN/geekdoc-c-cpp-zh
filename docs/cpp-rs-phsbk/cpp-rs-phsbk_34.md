# 迭代器和范围

> 原文：[`cel.cs.brown.edu/crp/idioms/iterators.html`](https://cel.cs.brown.edu/crp/idioms/iterators.html)

Rust 迭代器与 C++ [范围](https://en.cppreference.com/w/cpp/ranges.html) 类似，因为它们表示可迭序列，并且可以像使用范围视图一样进行操作。由于 C++范围是用迭代器定义的，而范围仅在 C++20 中引入，因此本章将 Rust 迭代器与 C++迭代器和 C++范围进行比较。

Rust 迭代器是单向迭代器，不是双向或随机访问迭代器。`Iterator`特质的定义反映了这一点：它所有的方法都是基于一个`next`方法，该方法返回一个包含迭代中下一个项目的`Option::Some`或`Option::None`。

Rust 迭代器也不像 C++迭代器与 C++ STL 算法库中的函数（如`std::sort`）一起使用时那样表示结构中的索引。

Rust 迭代器是输入迭代器还是输入/输出迭代器取决于迭代的项目是拥有值（输入）、引用（输入）还是可变引用（输入/输出）。迭代值的类型通常反映了正在迭代的结构是拥有值、引用还是可变引用。Rust 迭代器不能仅作为输出迭代器，因为迭代值必须始终初始化。

在某种意义上，Rust 的迭代器与 C++23 生成器非常相似（除了 Rust[尚未支持协程](https://github.com/rust-lang/rust/issues/43122)）。

## 迭代器、范围和`for`循环

在 C++中，任何具有`begin()`和`end()`方法以返回迭代器的对象（即，任何符合 C++20 `range`概念的模型）都可以与 for 循环一起使用。在 Rust 中，任何实现了`IntoIterator`特质的对象都可以与 for 循环一起使用。这包括迭代器本身，它们通过[泛型实现](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html)来实现该特质。

```rs
#include <iostream>
#include <vector>

int main() {
  std::vector<int> v{1, 2, 3};

  // prints 1, 2, 3
  for (auto &x : v) {
    std::cout << x << std::endl;
    x = x + 1;
  }

  // prints 2, 3, 4
  for (const auto &x : v) {
    std::cout << x << std::endl;
  }
} 
```

```rs

```

fn main() {

    let mut v = vec![1, 2, 3];

    // 打印 1, 2, 3

    for x in &mut v {

        println!("{}", x);

        *x = *x + 1;

    }

    // 打印 2, 3, 4

    for x in &v {

        println!("{}", x);

    }

}

```rs

```

在 C++和 Rust 中，迭代器都可以用于读取、写入或两者兼具。在 Rust 中，迭代器用于写入的使用取决于返回的元素类型。在上面的`Vec<i32>`的例子中，为`&mut Vec<i32>`实现的`IntoIterator`特质产生了一个包含可变引用`&mut i32`的迭代器，这使得可以修改向量中的值.^(1)

## 范围和视图

正如 [C++ 范围库](https://en.cppreference.com/w/cpp/ranges.html) 提供了许多定义管道的实用函数来转换范围一样，Rust 标准库定义了许多转换 Rust 迭代器的迭代器方法，包括将它们转换回像向量这样的集合。

```rs
#include <ranges>
#include <vector>

using namespace std::views;
using namespace std::ranges::views;

int main() {
 // clang-format off  // This example requires C++23
  auto v =
    iota(1)
      | filter([](int n) { return n % 2 == 1; })
      | transform([](int n) { return n + 3; })
      | take(10)
      | std::ranges::to<std::vector>();
 // clang-format on 
  // use v...
} 
```

```rs

```

fn main() {

    let v = (1..)

        .filter(|i| i % 2 == 1)

        .map(|i| i + 3)

        .take(10)

        .collect::<Vec<i32>>();

    // use v...

}

```rs

```

Rust 的 `collect` 迭代器方法可以将迭代器转换为实现了 `FromIterator` 的任何东西。如果 `v` 的类型可以从其后续使用中推断出来，则不需要在 `collect` 调用中指定类型。

在 C++ 和 Rust 中，视图或迭代器可以直接用作循环的值，而无需首先转换为类似向量这样的东西。同样，在这两种语言中，值的构造都是懒加载的。

```rs
#include <ranges>
#include <iostream>

using namespace std::views;
using namespace std::ranges::views;

int main() {
 // clang-format off    for (auto x :
        iota(1)
          | filter([](int n) { return n % 2 == 1; })
          | transform([](int n) { return n + 3; })
          | take(10)) {
      std::cout<< x << std::endl;
    }
 // clang-format on } 
```

```rs

```

fn main() {

    for x in (1..)

        .filter(|i| i % 2 == 1)

        .map(|i| i + 3)

        .take(10)

    {

        println!("{}", x);

    }

}

```rs

```

通过第三方 [itertools crate](https://docs.rs/itertools/latest/itertools/) 提供的迭代器附加有用方法，这些方法通过 扩展特质 实现。

## `IntoIterator` 和所有权

可以为类型 `T` 本身、引用 `&T` 或可变引用 `&mut T` 实现 `IntoIterator` 特质。迭代项的可能类型取决于特质的实现类型。例如，如果为 `&mut T` 实现，则通常项将是原始结构体仍拥有的项的可变引用。如果类型是 `T`，则项将是原始结构体中的所有项的所有权项.^(2)

由于循环中使用的结构体类型的行为取决于可能推断出来且因此不可见的类型，这可能导致令人惊讶的编译错误。特别是，遍历向量 `v` 而不是向量引用 `&v` 将消耗原始向量，使其不可访问。

在 C++ 中遍历结构体与在 Rust 中对可变引用调用 `into_iter` 最相似。

```rs

```

fn main() {

    let mut v = vec![

        String::from("a"),

        String::from("b"),

        String::from("c"),

    ];

    for x in &v {

        // x: &String

        println!("{}", x);

    }

    // 由于 v 被借用，而不是移动，因此它在这里仍然可访问。

    println!("{:?}", v);

    for x in &mut v {

        // x: &mut String

        x.push('!');

    }

    // 由于 v 被借用，而不是移动，因此它在这里仍然可访问。

    // 然而，v 的内容已被修改

    println!("{:?}", v);

    for x in v {

        // x: String

        // 每次迭代结束时释放 x

    }

    // v 不再可访问，因此这不会编译

    // println!("{:?}", v);

}

```rs

```

大多数可迭代类型也会提供专门用于访问引用或可变引用迭代器的方法。传统上，这些方法被称为 `iter` 和 `iter_mut`。在迭代不是立即与 for 循环一起使用，而是与其他迭代器方法一起使用的情况下，它们非常有用，因为引用操作符和方法调用的相对优先级。

## 为算法操作识别范围

C++ 使用迭代器来识别结构上 STL 算法库中函数应该操作的区域。Rust 迭代器不服务于这个目的。相反，有两种常见的替代方案。

第一种情况是，仅严格操作于前向迭代器的操作直接作用于迭代器。为了这个目的识别迭代器的特定部分可以通过使用迭代器方法来完成，例如 `take` 或 `filter` 方法。或者，对于某些类型，在转换为迭代器之前可以通过范围索引来获取切片（[slices can be taken by indexing with a range](https://doc.rust-lang.org/book/ch04-03-slices.html)）。

```rs
#include <algorithm>
#include <vector>

int main() {
  std::vector v{1, 2, 3, 4, 5, 6, 7, 8, 9};
  auto begin = v.begin() + 2;
  auto end = begin + 5;
  bool b(std::any_of(begin, end, [](int n) {
    return n % 2 == 0;
  }));
} 
```

```rs

```

fn main() {

    let v: Vec<i32> = (1..10).collect();

    let b = v

        .iter()

        .skip(2)

        .take(5)

        .any(|n| n % 2 == 0);

    // or

    let b2 = v[3..7].iter().any(|n| n % 2 == 0);

    // ...

}

```rs

```

第二种情况是，一些算法操作于切片。例如，Rust 标准库中的 [sort 方法](https://doc.rust-lang.org/std/primitive.slice.html#method.sort) 操作于切片。这类似于在 C++ 中，一个函数操作于 `std::span` 而不是迭代器。`Vec<T>` 上许多可用的方法实际上是在 `&[T]` 上定义的，并通过 [deref coercion](https://doc.rust-lang.org/book/ch15-02-deref.html) 在 `Vec<T>` 上提供。

```rs
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector v{9, 8, 7, 6, 5, 4, 3, 2, 1};

  for (auto n : v) {
    std::cout << n << ",";
  }
  std::cout << std::endl;

  std::sort(v.begin(), v.end());

  for (auto n : v) {
    std::cout << n << ",";
  }
  std::cout << std::endl;
} 
```

```rs

```

fn main() {

    let mut v: Vec<i32> = (1..10).rev().collect();

    println!("{:?}", v);

    v.sort();

    println!("{:?}", v);

}

```rs

```

## 迭代器失效

In C++, operations sometimes only invalidate some iterators on a value, such as the `erase` method on `std::vector` only invaliding iterators to the erased element and those after it, but not the ones before it.

在 Rust 中，迭代器借用整个迭代值的特性意味着在迭代过程中不能执行修改值本身的操作（例如从向量中删除值）。因此，在使用 Rust 迭代器时没有需要记住的迭代器失效规则。

然而，这也意味着在 C++ 中可以使用迭代器完成一些在 Rust 中无法完成的事情，例如在迭代过程中从向量中删除元素。相反，在 Rust 中有两种可能的方法：使用索引或使用辅助方法。

使用索引而不是迭代器会带来与 C++中相同的挑战，除了在安全 Rust 中，如果索引超出范围，程序会 panic 而不是执行未定义的行为。

使用辅助方法类似于通常推荐的针对较新 C++标准的编写建议。例如，在 Rust 中删除特定值的所有元素时，会使用`[`Vec::retain](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.retain)`方法，这类似于`std::vector`上的`remove_if`或`erase_if`，但带有负谓词。

```rs
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
  std::vector<int> v{1, 2, 3};

  auto newEnd = remove(v.begin(), v.end(), 2);
  v.erase(newEnd, v.end());

  // Or since C++20
  // std::erase(v, 2);

  for (auto x : v) {
    std::cout << x << std::endl;
  }
} 
```

```rs

```

fn main() {

    let mut v = vec![1, 2, 3];

    v.retain(|i| *i != 2);

    for x in &v {

        println!("{}", x);

    }

}

```rs

```

当使用迭代器时，会使用范围和视图部分中描述的方法。

## 实现 Rust 迭代器

本扩展示例定义了一个二叉树及其在树上的前序常量迭代器。模块结构包含在本示例中，因为定义私有项将是后续用于简化实现的模式的重要组成部分。

```rs
#include <memory>

template <typename V>
class Tree {
public:
    V value;
    std::unique_ptr<Tree<V>> left;
    std::unique_ptr<Tree<V>> right;
}; 
```

```rs

```

#![allow(unused)] fn main() { mod tree {

    /// 每个节点都有值的二叉树。

    /// 不一定是平衡的。

    pub struct Tree<V> {

        pub value: V,

        pub left: Option<Box<Tree<V>>>,

        pub right: Option<Box<Tree<V>>>,

    }

}

}

```rs

```

就像在 C++中迭代器和常量迭代器是不同的事物一样，在 Rust 中，对于所有者值、引用和可变引用有不同的迭代器。

例如，对于树类型`Tree<V>`，可能会提供以下方法以支持前序迭代。对于提供迭代引用的方法，引用是从原始结构中借用的，因此生命周期参数`'a`将引用与`self`中的项相关联。

|  | 方法 | 项目类型 |
| --- | --- | --- |
| 引用 | `fn preorder<'a>(&'a self) -> IterPreorder<'a, V>` | `&'a V` |
| 可变引用 | `fn preorder_mut<'a>(&'a mut self) -> IterMutPreorder<'a, V>` | `&'a mut V` |
| 所有者 | `fn into_preorder(self) -> IntoIterPreorder<V>` | `V` |

就像 C++迭代器一样，定义迭代器的基本复杂性在于确定如何捕获遍历结构的挂起状态。在这种情况下，挂起状态由要迭代的剩余树栈组成。

实现在提供的接口上有所不同。C++要求定义多个类型和方法，以便类型可以模拟前向迭代器。Rust 只需要定义将要迭代的元素类型和`next`方法。

```rs
#include <memory> #include <vector>   template <typename V>
class Tree {
public:
  class iterator {
    std::vector<const Tree<V> *> rest;

  public:
    using difference_type = long;
    using value_type = V;
    using pointer = const V *;
    using reference = const V &;
    using iterator_category =
        std::forward_iterator_tag;

    iterator() {}
    iterator(const Tree<V> *start) {
      rest.push_back(start);
    }

    reference operator*() const {
      return rest.back()->value;
    }

    iterator &operator++() {
      const Tree<V> *t = rest.back();
      rest.pop_back();
      if (t->right) {
        rest.push_back(t->right.get());
      }
      if (t->left) {
        rest.push_back(t->left.get());
      }
      return *this;
    }

    iterator operator++(int) {
      iterator retval = *this;
      const Tree<V> *t = rest.back();
      rest.pop_back();
      if (t->right) {
        rest.push_back(t->right.get());
      }
      if (t->left) {
        rest.push_back(t->left.get());
      }
      return retval;
    }

    bool operator==(const iterator &other) const {
      return rest == other.rest;
    }

    bool operator!=(const iterator &other) const {
      return !(*this == other);
    }
  };
  V value; std::unique_ptr<Tree<V>> left; std::unique_ptr<Tree<V>> right; }; 
```

```rs

```

#![allow(unused)] fn main() { mod tree {

/// 每个节点都有值的二叉树。 /// 不一定是平衡的。 pub struct Tree<V> { pub value: V, pub left: Option<Box<Tree<V>>>, pub right: Option<Box<Tree<V>>>, }      pub struct IterPreorder<'a, V>(Vec<&'a Tree<V>>);

    impl<'a, V> Iterator for IterPreorder<'a, V> {

        type Item = &'a V;

        // 这类似于一个组合

        // 运算符++ 和运算符*

        fn next(&mut self) -> Option<&'a V> {

            let Tree { value, left, right } = self.0.pop()?;

            if let Some(right) = right {

                self.0.push(right.as_ref());

            }

            if let Some(left) = left {

                self.0.push(left.as_ref());

            }

            Some(value)

        }

    }

}

}

```rs

```

剩下的步骤是将原始类型变为可迭代的。在 C++ 中，这涉及到定义 `begin` 和 `end` 方法。在 Rust 中，这涉及到实现一个显式产生迭代器的方法，或者实现 `IntoIterator` 特质。

当一个类型有多个可能的迭代方式且都不是规范方式时，省略 `IntoIterator` 特质实现是惯用的做法。省略实现需要用户有意选择要使用的迭代方式。以下提供了实现示例，但未排序的二叉树通常是省略特质实现的情况，这会强制用户在先序、后序和遍历顺序之间进行选择。

因为实现的迭代器是引用迭代器，所以特质的实现实际上是针对树的引用 `&Tree<V>`，而不是 `Tree<V>` 本身。

```rs
#include <iostream>
#include <memory> #include <vector> 
template <typename V>
class Tree {
public:
 class iterator { std::vector<const Tree<V> *> rest;   public: using difference_type = long; using value_type = V; using pointer = const V *; using reference = const V &; using iterator_category = std::forward_iterator_tag;   iterator() {} iterator(const Tree<V> *start) { rest.push_back(start); }   reference operator*() const { return rest.back()->value; }   iterator &operator++() { const Tree<V> *t = rest.back(); rest.pop_back(); if (t->right) { rest.push_back(t->right.get()); } if (t->left) { rest.push_back(t->left.get()); } return *this; }   iterator operator++(int) { iterator retval = *this; const Tree<V> *t = rest.back(); rest.pop_back(); if (t->right) { rest.push_back(t->right.get()); } if (t->left) { rest.push_back(t->left.get()); } return retval; }   bool operator==(const iterator &other) const { return rest == other.rest; }   bool operator!=(const iterator &other) const { return !(*this == other); } };    iterator begin() const {
    return iterator(this);
  }

  iterator end() const {
    return iterator();
  }
  V value; std::unique_ptr<Tree<V>> left; std::unique_ptr<Tree<V>> right; };

int main() {
  Tree<int> t{1,
              std::make_unique<Tree<int>>(
                  2, nullptr, nullptr),
              std::make_unique<Tree<int>>(
                  3,
                  std::make_unique<Tree<int>>(
                      4, nullptr, nullptr),
                  nullptr)};

  for (auto v : t) {
    std::cout << v << std::endl;
  }
} 
```

```rs

```

mod tree {

/// 每个节点都有值的二叉树。不一定平衡。 pub struct Tree<V> { pub value: V, pub left: Option<Box<Tree<V>>>, pub right: Option<Box<Tree<V>>>, }   pub struct IterPreorder<'a, V>(Vec<&'a Tree<V>>);   impl<'a, V> Iterator for IterPreorder<'a, V> { type Item = &'a V; fn next(&mut self) -> Option<&'a V> { match self.0.pop() { None => None, Some(t) => { let Tree { value, left, right } = t; if let Some(right) = right { self.0.push(right.as_ref()); } if let Some(left) = left { self.0.push(left.as_ref()); } Some(value) } } } }      impl<V> Tree<V> {

        pub fn preorder(&self) -> IterPreorder<V> {

            IterPreorder(vec![self])

        }

    }

    impl<'a, V> IntoIterator for &'a Tree<V> {

        type Item = &'a V;

        type IntoIter = IterPreorder<'a, V>;

        fn into_iter(self) -> Self::IntoIter {

            self.preorder()

        }

    }

}

use tree::*;

fn main() {

    let t = Tree {

        value: 1,

        left: Some(Box::new(Tree {

            value: 2,

            left: None,

            right: None,

        })),

        right: Some(Box::new(Tree {

            value: 3,

            left: Some(Box::new(Tree {

                value: 4,

                left: None,

                right: None,

            })),

            right: None,

        })),

    };

    for n in t.preorder() {

        println!("{}", n);

    }

    for n in &t {

        println!("{}", n);

    }

}

```rs

```

对于可变引用和所有权的迭代器实现可以类似地进行。对于这三种情况，都有三个 `IntoIterator` 实现，一个用于 `&Tree<V>`，一个用于 `&mut Tree<V>`，一个用于 `Tree<V>`。

### 减少代码重复

与在 C++ 中实现迭代器和常量迭代器类似，在 Rust 中实现所有权的迭代器、引用和可变引用的迭代器可能会导致代码重复。

一种解决这个问题的模式是定义一个私有特质来捕获类型的分解，然后通过该特质的泛型实现来通过`Iterator`特质实现。然后可以使用包装结构体来暴露迭代行为而不暴露辅助特质。

以下示例实现了上述`Tree<V>`类型的模式，作为上述更简单但更冗余方法的替代方案。

```rs

```

mod tree {

    /// 每个节点都有值的二叉树。

    /// 不一定是平衡的。

    pub struct Tree<V> {

        pub value: V,

        pub left: Option<Box<Tree<V>>>,

        pub right: Option<Box<Tree<V>>>,

    }

    impl<V> Tree<V> {

        // ... 静态方法用于构建树 ...

/// 使用两个子树构建一个新的节点。 pub fn node(value: V, left: Tree<V>, right: Tree<V>) -> Tree<V> { Tree { value, left: Some(Box::new(left)), right: Some(Box::new(right)), } }   /// 使用左子树构建一个新的节点。 pub fn left(value: V, left: Tree<V>) -> Tree<V> { Tree { value, left: Some(Box::new(left)), right: None, } }   /// 使用右子树构建一个新的节点。 pub fn right(value: V, right: Tree<V>) -> Tree<V> { Tree { value, left: None, right: Some(Box::new(right)), } }   /// 构建一个新的叶节点。 pub fn leaf(value: V) -> Self { Tree { value, left: None, right: None, } }    }

    /// 用于抽象访问的内部特质

    /// 到树组件。

    ///

    /// 这减少了代码重复。

    /// 实现本质上

    /// 对于 Tree<V>，&Tree<V>，以及&mut Tree<V>也是同样的。

    /// 和 Tree<V>，&Tree<V>，以及&mut Tree<V>。

    trait Treeish: Sized {

        type Output;

        fn get(self) -> (Option<Self>, Self::Output, Option<Self>);

    }

    impl<V> Treeish for Tree<V> {

        type Output = V;

        fn get(self) -> (Option<Self>, Self::Output, Option<Self>) {

            let Tree { value, left, right } = self;

            (left.map(|x| *x), value, right.map(|x| *x))

        }

    }

    impl<'a, V> Treeish for &'a Tree<V> {

        type Output = &'a V;

        fn get(self) -> (Option<Self>, Self::Output, Option<Self>) {

            let Tree { value, left, right } = self;

            (left.as_deref(), value, right.as_deref())

        }

    }

    impl<'a, V> Treeish for &'a mut Tree<V> {

        type Output = &'a mut V;

        fn get(self) -> (Option<Self>, Self::Output, Option<Self>) {

            let Tree { value, left, right } = self;

            (left.as_deref_mut(), value, right.as_deref_mut())

        }

    }

    /// 用于实现 Iterator 的内部结构体

    /// 在 Treeish 的术语中。

    struct Preorder<T>(Vec<T>);

    impl<T> Iterator for Preorder<T>

    where

        T: Treeish,

    {

        type Item = T::Output;

        fn next(&mut self) -> Option<Self::Item> {

            let next = self.0.pop();

            match next {

                None => None,

                Some(t) => {

                    // 这里使用了辅助特质

                    let (left, value, right) = t.get();

                    if let Some(right) = right {

                        self.0.push(right);

                    }

                    if let Some(left) = left {

                        self.0.push(left);

                    }

                    Some(value)

                }

            }

        }

    }

    // 用于暴露迭代器的包装器。包装器是必要的

    // 为了保持 Treeish 私有。Treeish::Output 将

    // 否则将暴露出来，因此需要将 Treeish 设置为 public。

    /// 前序迭代器

    pub struct IntoIterPreorder<V>(Preorder<Tree<V>>);

    /// 前序迭代器

    pub struct IterPreorder<'a, V>(Preorder<&'a Tree<V>>);

    /// 前序迭代器

    pub struct IterMutPreorder<'a, V>(Preorder<&'a mut Tree<V>>);

    // 委托给包装的实现。

    impl<V> Iterator for IntoIterPreorder<V> {

        type Item = V;

        fn next(&mut self) -> Option<Self::Item> {

            self.0.next()

        }

    }

    impl<'a, V> Iterator for IterPreorder<'a, V> {

        type Item = &'a V;

        fn next(&mut self) -> Option<Self::Item> {

            self.0.next()

        }

    }

    impl<'a, V> Iterator for IterMutPreorder<'a, V> {

        type Item = &'a mut V;

        fn next(&mut self) -> Option<Self::Item> {

            self.0.next()

        }

    }

    impl<V> Tree<V> {

        pub fn preorder(self) -> IntoIterPreorder<V> {

            IntoIterPreorder(Preorder(vec![self]))

        }

        pub fn preorder_ref(&self) -> IterPreorder<V> {

            IterPreorder(Preorder(vec![self]))

        }

        pub fn preorder_ref_mut(&mut self) -> IterMutPreorder<V> {

            IterMutPreorder(Preorder(vec![self]))

        }

    }

}

use tree::*;

fn main() {

    let mut t = Tree::node(

        0,

        Tree::left(1, Tree::leaf(2)),

        Tree::node(3, Tree::leaf(4), Tree::right(5, Tree::leaf(6))),

    );

    for x in t.preorder_ref_mut() {

        *x += 10;

    }

    for x in t.preorder_ref() {

        println!("{}", x);

    }

}

```rs

```

## 双向和随机访问迭代器

Rust 标准库不包含双向或随机访问迭代器的支持。对于这些迭代器支持的大多数用例，使用数字索引就足够了。

标准库确实支持[双端迭代器](https://doc.rust-lang.org/std/iter/trait.DoubleEndedIterator.html)，这允许从迭代器的末尾消费项目。然而，每个项目仍然只能消费一次：当前端和后端在中部相遇时，迭代结束。

<link rel="stylesheet" type="text/css" href="../quiz/style.css">

* * *

1.  可变引用的安全性由以下事实给出：引用从向量借用，不重叠，并且迭代器不会多次产生。 ↑

1.  拥有的项目本身可能是引用。例如，在`Vec<&str>`上调用`into_iter`不会导致迭代`String`值，即使该向量本身被消费。↑

[在此处点击以给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Iterators and ranges)
