# 对象身份

> 原文：[`cel.cs.brown.edu/crp/idioms/object_identity.html`](https://cel.cs.brown.edu/crp/idioms/object_identity.html)

在 C++ 中，对象的指针有时被用来表示程序逻辑中的其身份。

在某些情况下，这是一种标准的优化，例如在实现复制赋值运算符时。

在其他情况下，指针值被用作逻辑身份来区分具有相同属性的对象的具体实例。例如，表示一个可能具有不同标签的节点的带标签图。

In Rust，这些情况中的一些不适用，而其他情况通常通过实现值的合成身份概念来处理。

## 重载复制赋值和相等比较运算符

例如，在实现复制赋值运算符时，如果复制的对象和被赋值的对象是相同的，可能会短路。注意，在这种情况下，指针值不会被存储。

在实现 Rust 的复制赋值运算符等价物 `Clone::clone_from` 时，这种优化是不必要的。`Clone::clone_from` 的类型阻止同一个对象作为两个参数传递，因为其中一个参数是一个可变引用，它是排他的，因此阻止另一个引用参数引用同一个对象。

```rs
struct Person
{
    std::string name;
    // many other expensive-to-copy fields

    Person& operator=(const Person& other) {
        // compare object identity first
        if (this != &other) {
            this.name = other.name;
            // copy the other expensive-to-copy fields
        }

        return *this;
    }
}; 
```

```rs

```

#![allow(unused)] fn main() { struct Person {

    name: String,

}

实现对 `Person` 的 `Clone` trait {

    fn clone(&self) -> Self {

        Self { name: self.name.clone() }

    }

    fn clone_from(&mut self, source: &Self) {

        // self 和 source 不能在这里相同，

        // because that would mean there are a

        // mutable and an immutable reference to

        // the same memory location. Therefore, a

        // check for assignment to self is not

        // needed, even for the purpose of

        // optimization.

        self.name.clone_from(&source.name);

    }

}

}

```rs

```

在 C++ 中，大多数比较都是在对象与其自身之间进行的（例如，对象的主要用途是存储在哈希集中），并且比较不等价对象是昂贵的，因此可能会使用对象身份比较作为相等比较运算符重载的优化。

为了在 Rust 中支持类似操作，可以使用 `std::ptr::eq`(https://doc.rust-lang.org/std/ptr/fn.eq.html)。

```rs
struct Person
{
    std::string name;
    // many other expensive-to-compare fields
};

bool operator==(const Person& lhs, const Person& rhs) {
    // compare object identity first
    if (&lhs == &rhs) {
        return true;
    }

    // compare the other expensive-to-compare fields

    return true;
} 
```

```rs

```

#![allow(unused)] fn main() { struct Person {

    name: String,

    // many other expensive-to-compare fields

}

实现对 `Person` 的 `PartialEq` trait {

    fn eq(&self, other: &Self) -> bool {

        if std::ptr::eq(self, other) {

            return true;

        }

        // compare other expensive-to-compare fields

        true

    }

}

实现对 `Person` 的 `Eq` trait {}

}

```rs

```

## 在关系结构中区分值

另一种用法是当关系值使用外部于值的数据结构表示时，例如在表示一个标签图时，其中多个节点可能共享相同的标签，但与其他节点集之间有边。这与早期情况不同，因为指针值被保留。

一个现实世界的例子是在 LLVM 代码库中，其中 AST 中的声明、语句和表达式的出现通过对象标识来区分。例如，变量表达式（`class DeclRefExpr`）包含变量引用的声明出现的[指针](https://github.com/llvm/llvm-project/blob/ddc48fefe389789f64713b5924a03fb2b7961ef3/clang/include/clang/AST/Expr.h#L1265C1-L1275C16)。

类似地，当比较两个变量声明是否代表同一变量的声明时，[使用一些规范 `VarDecl` 的指针](https://github.com/llvm/llvm-project/blob/aa33c095617400a23a2b814c4defeb12e7761639/clang/lib/AST/Stmt.cpp#L1476-L1485)：

```rs
VarDecl *VarDecl::getCanonicalDecl();

bool CapturedStmt::capturesVariable(const VarDecl *Var) const {
  for (const auto &I : captures()) {
    if (!I.capturesVariable() && !I.capturesVariableByCopy())
      continue;
    if (I.getCapturedVar()->getCanonicalDecl() == Var->getCanonicalDecl())
      return true;
  }

  return false;
} 
```

这种用法在 C++ 中通常是不被鼓励的，因为存在使用后释放的漏洞风险，但在性能敏感的应用程序中可能会使用，在这些应用程序中，存储表示映射的内存或从标识符解析实体值所需的额外间接引用的成本是过高的。

在 Rust 中，通常更喜欢使用合成标识符来表示对象的标识。这在一定程度上是一种用于建模自引用数据结构的技巧。

例如，一个流行的 Rust 图库 [petgraph](https://docs.rs/petgraph/latest/petgraph/) 使用 `u32` 作为其默认节点标识类型。这导致了额外的调用以解引用合成标识符到表示节点的标签，以及存储节点到标签映射所需的额外内存。

使用相同的合成标识符技术的一个简化的图表示如下，它通过表示标签和边的向量的索引来表示节点标识。

```rs

```

#![allow(unused)] fn main() { enum Color {

    Red,

    Blue

}

struct Graph {

    /// 从节点 ID 到节点标签的映射，在这里是颜色。

    nodes_labels: Vec<Color>,

    /// 从节点 ID 到相邻节点 ID 的映射。

    edges: Vec<Vec<usize>>,

}

}

```rs

```

如果性能要求使得使用合成标识符不可接受，那么可能需要使用防止值移动。可以使用 `Pin` 和 `PhantomPinned` 结构体（[链接](https://doc.rust-lang.org/std/pin/index.html)）来实现类似于在 C++ 中删除移动构造函数的效果。

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Object identity)
