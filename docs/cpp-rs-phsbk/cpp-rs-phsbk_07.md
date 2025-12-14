# 析构函数和资源清理

> 原文：[`cel.cs.brown.edu/crp/idioms/destructors.html`](https://cel.cs.brown.edu/crp/idioms/destructors.html)

在 C++中，类`T`的析构函数通过提供一个特殊的成员函数`~T()`来定义。要在 Rust 中实现等效功能，需要对类型实现`Drop`特质。

例如，请参阅关于复制和移动构造函数的章节。

`Drop`实现对于管理资源的类型在 Rust 中扮演着与 C++中析构函数相同的角色。也就是说，它们允许在值的生命周期结束时清理由值拥有的资源。

在 Rust 中，值的`Drop::drop`方法在拥有该值的变量超出作用域时由析构函数自动调用。与 C++不同，不能手动调用析构方法。相反，自动的“析构粘合剂”隐式调用字段的析构函数。

## 生命周期、析构函数和销毁顺序

C++析构函数在变量超出作用域时按构造顺序的相反顺序调用，或者在动态分配的对象被删除时调用。这包括从移动对象中调用的析构函数。

在 Rust 中，超出作用域的项目删除顺序与 C++类似（声明顺序的相反顺序）。如果需要关于删除顺序的更详细信息（例如，用于编写不安全代码），则在[语言参考](https://doc.rust-lang.org/reference/destructors.html)中描述了删除顺序的完整规则。然而，在 Rust 中移动对象不会留下一个被移动的对象，该对象将调用析构函数。

```rs
#include <iostream>
#include <utility>

struct A {
  int id;

  A(int id) : id(id) {}

  // copy constructor
  A(A &other) : id(other.id) {}

  // move constructor
  A(A &&other) : id(other.id) {
    other.id = 0;
  }

  // destructor
  ~A() {
    std::cout << id << std::endl;
  }
};

int accept(A x) {
  return x.id;
} // the destructor of x is called after the
  // return expression is evaluated

// Prints:
// 2
// 3
// 0
// 1
int main() {
  A x(1);
  A y(2);

  accept(std::move(y));

  A z(3);

  return 0;
} 
```

```rs

```

struct A {

    id: i32,

}

impl Drop for A {

    fn drop(&mut self) {

        println!("{}", self.id)

    }

}

fn accept(x: A) -> i32 {

    return x.id;

}

// 打印：

// 2

// 3

// 1

fn main() {

    let x = A { id: 1 };

    let y = A { id: 2 };

    accept(y);

    let z = A { id: 3 };

}

```rs

```

在 Rust 中，当`y`的所有权被移动到函数`accept`中后，没有剩余的对象，因此没有额外的`Drop::drop`调用（在 C++示例中打印`0`）。

Rust 的析构方法在离开作用域时确实会运行，尽管在响应初始 panic 而调用的析构函数中 panic 不会运行。

Rust 中字段的删除顺序本质上与 C++中非静态类成员的删除顺序相反。再次强调，Rust 析构函数中具体发生的事情在[语言参考](https://doc.rust-lang.org/reference/destructors.html#r-destructors.operation)中有详细说明。

```rs
#include <iostream>
#include <string>

struct Part {
  std::string name;

  ~Part() {
    std::cout << "Dropped " << name << std::endl;
  }
};

struct Widget {
  Part part1;
  Part part2;
  Part part3;
};

int main() {
  Widget w{"1", "2", "3"};
  // Prints:
  // 3
  // 2
  // 1
} 
```

```rs

```

struct Part(&'static str);

impl Drop for Part {

    fn drop(&mut self) {

        println!("{}", self.0);

    }

}

struct Widget {

    part1: Part,

    part2: Part,

    part3: Part,

}

fn main() {

    let w = Widget {

        part1: Part("1"),

        part2: Part("2"),

        part3: Part("3"),

    };

    // 打印：

    // 1

    // 2

    // 3

}

```rs

```

## 早期清理和显式销毁值

在 C++ 中，你可以显式地销毁一个对象。这主要用于那些使用了 placement new 在特定内存位置分配对象的情况，因此析构函数不会被隐式调用。

然而，一旦显式调用了析构函数，[它可能不会被再次调用，即使是隐式调用](https://eel.is/c++draft/class.dtor#note-8)。因此，析构函数不能用于早期清理。相反，类必须设计一个单独的清理方法来释放资源，但对象的状态仍然允许析构函数被调用，或者使用对象的函数必须被结构化，以便变量在期望的时间出作用域。

在 Rust 中，可以使用 `std::mem::drop` 来提前释放值，以便进行早期清理。这是因为在非 `Copy` 类型的情况下（对于简单可复制的类型），对象的拥有权实际上转移到了 `std::mem::drop` 函数，因此当参数的生命周期结束时，会在 `std::mem::drop` 的末尾调用 `Drop::drop`。

因此，`std::mem::drop` 可以用于在不重新结构化函数以强制变量提前出作用域的情况下，对资源进行早期清理。

例如，以下代码在堆上分配了一个大向量，但在分配第二个大向量之前显式地释放了它，从而减少了总的内存使用。

```rs

```

fn main() {

    let v = vec![0u32; 100000];

    // ... 使用 v

    std::mem::drop(v);

    // 在这里不能再使用 v

    let v2 = vec![0u32; 100000];

    // ... 使用 v2

}

```rs

```

<link rel="stylesheet" type="text/css" href="../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Destructors and resource cleanup)
