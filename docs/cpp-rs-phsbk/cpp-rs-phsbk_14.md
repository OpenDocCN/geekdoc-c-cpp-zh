# 模板类、函数和方法

> 原文：[`cel.cs.brown.edu/crp/idioms/data_modeling/templates.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/templates.html)

C++中模板最常见的用途是定义适用于任何类型（或至少适用于提供某些方法的任何类型）的类、方法或函数。这种用法在 STL 的容器类（如`<vector>`）和算法库（`<algorithm>`）中很常见。

以下示例定义了一个模板，用于表示为邻接表形式的定向图，其中图在节点标签的类型上是通用的。尽管示例展示了模板类，但与 Rust 的模板方法和模板函数相同的比较也适用于 Rust。

在上述示例中演示的使用案例中，使用 C++模板定义类和使用 Rust 泛型定义结构体之间的实际差异很少。当在 C++中使用接受`typename`或`class`参数的模板时，可以在 Rust 中相应地使用类型参数。

```rs
#include <stdexcept>
#include <vector>

template <typename Label>
class DirectedGraph {
  std::vector<std::vector<size_t>> adjacencies;
  std::vector<Label> nodeLabels;

public:
  size_t addNode(Label label) {
    adjacencies.push_back(std::vector<size_t>());
    nodeLabels.push_back(std::move(label));
    return numNodes() - 1;
  }

  void addEdge(size_t from, size_t to) {
    size_t numNodes = this->numNodes();
    if (from >= numNodes || to >= numNodes) {
      throw std::invalid_argument(
          "Node index out of range");
    }
    adjacencies[from].push_back(to);
  }

  size_t numNodes() const {
    return adjacencies.size();
  }
}; 
```

```rs

```

#![allow(unused)] fn main() { pub struct DirectedGraph<Label> {

    adjacencies: Vec<Vec<usize>>,

    node_labels: Vec<Label>,

}

impl<Label> DirectedGraph<Label> {

    pub fn new() -> Self {

        DirectedGraph {

            adjacencies: Vec::new(),

            node_labels: Vec::new(),

        }

    }

    pub fn add_node(

        &mut self,

        label: Label,

    ) -> usize {

        self.adjacencies.push(Vec::new());

        self.node_labels.push(label);

        self.num_nodes() - 1

    }

    pub fn add_edge(

        &mut self,

        from: usize,

        to: usize,

    ) -> Result<(), &str> {

        let num_nodes = self.num_nodes();

        if from >= num_nodes || to >= num_nodes {

            Err("Node index out of range.")

        } else {

            self.adjacencies[from].push(to);

            Ok(())

        }

    }

    pub fn num_nodes(&self) -> usize {

        self.node_labels.len()

    }

}

}

```rs

```

在上述示例中演示的使用案例中，使用 C++模板定义类和使用 Rust 泛型定义结构体之间的实际差异很少。当在 C++中使用接受`typename`或`class`参数的模板时，可以在 Rust 中相应地使用类型参数。

## 参数化类型的操作

当尝试对值执行操作时，差异变得更加明显。以下代码列表向 Rust 和 C++示例中添加了一个获取图中最小节点的方法。

```rs
#include <optional>
#include <stdexcept> #include <vector> 
template <typename Label>
class DirectedGraph {
 std::vector<std::vector<size_t>> adjacencies; std::vector<Label> nodeLabels;   public:
 size_t addNode(Label label) { adjacencies.push_back(std::vector<size_t>()); nodeLabels.push_back(std::move(label)); return numNodes() - 1; }   void addEdge(size_t from, size_t to) { size_t numNodes = this->numNodes(); if (from >= numNodes || to >= numNodes) { throw std::invalid_argument( "Node index out of range"); } adjacencies[from].push_back(to); }   size_t numNodes() const { return adjacencies.size(); }    std::optional<size_t> smallestNode() {
    if (nodeLabels.empty()) {
      return std::nullopt;
    }
    Label &least = nodeLabels[0];
    size_t index = 0;

    for (int i = 1; i < nodeLabels.size(); i++) {
      if (least > nodeLabels[i]) {
        least = nodeLabels[i];
        index = i;
      }
    }
    return std::optional(index);
  }
}; 
```

```rs

```

#![allow(unused)] fn main() { pub struct DirectedGraph<Label> {

adjacencies: Vec<Vec<usize>>, node_labels: Vec<Label>, }   impl<Label> DirectedGraph<Label> {

pub fn new() -> Self { DirectedGraph { adjacencies: Vec::new(), node_labels: Vec::new(), } }   pub fn add_node( &mut self, label: Label, ) -> usize { self.adjacencies.push(Vec::new()); self.node_labels.push(label); self.num_nodes() - 1 }   pub fn num_nodes(&self) -> usize { self.node_labels.len() }   pub fn add_edge( &mut self, from: usize, to: usize, ) -> Result<(), &str> { if from > self.num_nodes() || to > self.num_nodes() { Err("Node not in graph.") } else { self.adjacencies[from].push(to); Ok(()) } }    pub fn smallest_node(&self) -> Option<usize>

    where

        Label: Ord,

    {

        // 与 C++匹配，但不是惯用的

        // implementation!

        if self.node_labels.is_empty() {

            None

        } else {

            let mut least = &self.node_labels[0];

            let mut index = 0;

            for i in 1..self.node_labels.len() {

                if *least > self.node_labels[i] {

                    least = &self.node_labels[i];

                    index = i;

                }

            }

            Some(index)

        }

    }

}

}

```rs

```

这些实现之间的主要区别在于，在 C++ 版本中，`>` 操作符或 `operator>` 方法在不知道是否为该类型定义的情况下用于值。在 Rust 版本中，有一个约束要求 `Label` 类型实现 `Ord` 特性。（有关 Rust 特性和它们如何与 C++ 概念相关的更多详细信息，请参阅概念、接口和静态分发章节。）

与 C++ 模板不同，Rust 中的泛型定义在定义点而不是使用点进行类型检查。这意味着要对类型参数的类型进行约束，以便在值上使用操作。如上述示例所示，与 C++ 概念和 `requires` 类似，约束可以要求对单个方法而不是整个泛型类进行要求。

在 Rust 中，将特质的界限放在需要界限的具体事物上是一种最佳实践，以便使类型的整体使用更加灵活。

作为旁注，`smallest_node` 的更惯用实现利用了 Rust 的迭代器。这种实现风格可能需要程序员适应，尤其是那些更习惯于早期示例中使用的实现风格的程序员。

```rs

```

#![allow(unused)] fn main() { pub struct DirectedGraph<Label> {

adjacencies: Vec<Vec<usize>>, node_labels: Vec<Label>, }   impl<Label> DirectedGraph<Label> {

pub fn new() -> Self { DirectedGraph { adjacencies: Vec::new(), node_labels: Vec::new(), } }   pub fn add_node( &mut self, label: Label, ) -> usize { self.adjacencies.push(Vec::new()); self.node_labels.push(label); self.num_nodes() - 1 }   pub fn num_nodes(&self) -> usize { self.node_labels.len() }   pub fn add_edge( &mut self, from: usize, to: usize, ) -> Result<(), &str> { if from > self.num_nodes() || to > self.num_nodes() { Err("Node not in graph.") } else { self.adjacencies[from].push(to); Ok(()) } }    pub fn smallest_node(&self) -> Option<usize>

    where

        Label: Ord,

    {

        self.node_labels

            .iter()

            .enumerate()

            .map(|(i, l)| (l, i))

            .min()

            .map(|(_, i)| i)

    }

}

}

```rs

```

更为惯用的实现会利用 [itertools crate](https://docs.rs/itertools/latest/itertools/trait.Itertools.html#method.position_min)。

```rs
use itertools::*;

pub struct DirectedGraph<Label> {
 adjacencies: Vec<Vec<usize>>, node_labels: Vec<Label>, }   impl<Label> DirectedGraph<Label> {
 pub fn new() -> Self { DirectedGraph { adjacencies: Vec::new(), node_labels: Vec::new(), } }   pub fn add_node( &mut self, label: Label, ) -> usize { self.adjacencies.push(Vec::new()); self.node_labels.push(label); self.num_nodes() - 1 }   pub fn num_nodes(&self) -> usize { self.node_labels.len() }   pub fn add_edge( &mut self, from: usize, to: usize, ) -> Result<(), &str> { if from > self.num_nodes() || to > self.num_nodes() { Err("Node not in graph.") } else { self.adjacencies[from].push(to); Ok(()) } }      pub fn smallest_node(&self) -> Option<usize>
    where
        Label: Ord,
    {
        self.node_labels.iter().position_min()
    }
}
```

## `constexpr` 模板参数

Rust 还支持 `constexpr` 模板参数的等效功能。例如，可以定义一个泛型函数，该函数返回从特定值开始的连续整数数组，其大小在编译时确定。

```rs
#include <array>
#include <cstddef>

template <size_t N>
std::array<int, N>
makeSequentialArray(int start) {
  std::array<int, N> arr;
  for (size_t i = 0; i < N; i++) {
    arr[i] = start + i;
  }
} 
```

```rs

```

#![allow(unused)] fn main() { fn make_sequential_array<const N: usize>(

    start: i32,

) -> [i32; N] {

    std::array::from_fn(|i| start + i as i32)

}

}

```rs

```

对应的 Rust 函数使用辅助函数 `std::array::from_fn` 来构建数组。`from_fn` 本身将元素类型和常量作为类型参数。这些参数被省略，因为 Rust 可以推断它们，因为它们都是产生数组类型的一部分。

## Rust 的 `Self` 类型

在 Rust 结构定义、`impl` 块或 `impl` 特质块内部，存在一个 `Self` 类型，该类型在作用域内。`Self` 类型是所有泛型类型参数都已填充的正在被定义的类的类型。在有许多参数且否则必须列出的情况下，引用此类型可能很有用。

在定义泛型特质时，`Self` 类型是必要的，以引用具体的实现类型。因为 Rust 在具体类型之间没有继承，也没有方法重写，所以这足以避免需要将实现类型作为类型参数传递。

关于此的示例，请参阅 古怪重复出现的模板模式 章节。

## 关于类型检查和类型错误的说明

在定义点而不是模板展开点检查泛型类型会影响错误检测的时间和错误报告的方式。一些这种差异不能通过一致地使用 C++ 概念来声明所需的操作来实现。

例如，一个人可能会不小心将 `nodeLabels` 成员变量做成 `size_t` 类型的向量而不是标签参数的向量。如果用于图的测试用例中使用的标签类型都可以转换为整数，则不会检测到错误。

一个类似的 Rust 程序无法编译，即使没有实例化泛型结构的具体类型的函数。

```rs
#include <stdexcept>
#include <vector>

template <typename Label>
class DirectedGraph {
  std::vector<std::vector<size_t>> adjacencies;
  // The mistake is here: size_t should be Label
  std::vector<size_t> nodeLabels;

public:
  Label getNode(size_t nodeId) {
    return nodeLabels[nodeId];
  }

  size_t addNode(Label label) {
    adjacencies.push_back(std::vector<size_t>());
    nodeLabels.push_back(std::move(label));
    return numNodes() - 1;
  }

  size_t numNodes() const {
    return adjacencies.size();
  }
};

#define BOOST_TEST_MODULE DirectedGraphTests
#include <boost/test/included/unit_test.hpp>

BOOST_AUTO_TEST_CASE(test_add_node_int) {
  DirectedGraph<int> g;
  auto n1 = g.addNode(1);
  BOOST_CHECK_EQUAL(1, g.getNode(n1));
}

BOOST_AUTO_TEST_CASE(test_add_node_float) {
  DirectedGraph<float> g;
  float label = 1.0f;
  auto n1 = g.addNode(label);
  BOOST_CHECK_CLOSE(label, g.getNode(n1), 0.0001);
} 
```

```rs
pub struct DirectedGraph<Label> {
    adjacencies: Vec<Vec<usize>>,
    // The mistake is here: size_t should be Label
    node_labels: Vec<usize>,
}

impl<Label> DirectedGraph<Label> {
    pub fn new() -> Self {
        DirectedGraph {
            adjacencies: Vec::new(),
            node_labels: Vec::new(),
        }
    }

    pub fn get_node(
        &self,
        node_id: usize,
    ) -> Option<&Label> {
        self.node_labels.get(node_id)
    }

    pub fn add_node(
        &mut self,
        label: Label,
    ) -> usize {
        self.adjacencies.push(Vec::new());
        self.node_labels.push(label);
        self.num_nodes() - 1
    }

    pub fn num_nodes(&self) -> usize {
        self.node_labels.len()
    }
}
```

尽管有错误，C++ 示例仍然可以编译并通过测试。

```rs
Running 2 test cases...

*** No errors detected 
```

即使没有测试用例，Rust 示例也无法编译并产生用于识别错误的有用信息。

```rs
error[E0308]: mismatched types
    --> example.rs:26:31
     |
6    | impl<Label> DirectedGraph<Label> {
     |      ----- found this type parameter
...
26   |         self.node_labels.push(label);
     |                          ---- ^^^^^ expected `usize`, found type parameter `Label`
     |                          |
     |                          arguments to this method are incorrect
     |
     = note:        expected type `usize`
             found type parameter `Label` 
```

## 生命周期参数

Rust 的泛型也用于类、方法、特质和函数，这些函数在它们操作的引用的生命周期中是泛型的。与其他类型参数不同，使用具有不同生命周期的函数不会在编译代码中生成函数的额外副本，因为生命周期不会影响运行时表示。

概念章节包括 Rust 泛型与生命周期交互的示例。

## 条件编译

C++模板与 Rust 泛型之间一个显著的区别是，C++模板实际上是一种更通用的宏语言，支持诸如条件编译（例如，与`if constexpr`、`requires`或`std::enable_if`结合使用时）等功能。Rust 通过其宏系统支持这些用例，该系统与 C++有显著不同。宏系统最常见的使用，即条件编译，由[`cfg`属性和`cfg!`宏](https://doc.rust-lang.org/rust-by-example/attribute/cfg.html)提供。

Rust 中将条件编译与泛型分离的设计考虑与从 Rust 中省略模板特化的设计考虑相似。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为我们提供关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=模板类、函数和方法)
