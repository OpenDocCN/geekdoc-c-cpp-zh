# 复制和移动构造函数

> 原文：[`cel.cs.brown.edu/crp/idioms/constructors/copy_and_move_constructors.html`](https://cel.cs.brown.edu/crp/idioms/constructors/copy_and_move_constructors.html)

在 C++ 和 Rust 中，通常很少需要手动编写复制或移动构造函数（或它们的 Rust 等价物）。在 C++ 中，这是因为隐式定义对于大多数目的来说已经足够好了，尤其是在使用智能指针（即遵循[零规则](https://en.cppreference.com/w/cpp/language/rule_of_three)）时。在 Rust 中，这是因为移动语义是默认的，自动推导的 `Clone` 和 `Copy` 特性的实现对于大多数目的来说已经足够好了。

对于以下 C++ 类，隐式定义的复制和移动构造函数已经足够。在 Rust 中的等效实现使用标准库提供的 derive 宏来实现相应的特性。

```rs
#include <memory> #include <string>   struct Age {
  unsigned int years;

  Age(unsigned int years) : years(years) {}

  // copy and move constructors and destructor
  // implicitly declared and defined
};

struct Person {
  Age age;
  std::string name;
  std::shared_ptr<Person> best_friend;

  Person(Age age,
         std::string name,
         std::shared_ptr<Person> best_friend)
      : age(std::move(age)), name(std::move(name)),
        best_friend(std::move(best_friend)) {}

  // copy and move constructors and destructor
  // implicitly declared and defined
}; 
```

```rs

```

#![allow(unused)] fn main() { use std::rc::Rc;

#[derive(Clone, Copy)]

struct Age {

    years: u32,

}

#[derive(Clone)]

struct Person {

    age: 年龄,

    name: String,

    best_friend: Rc<Person>,

}

}

```rs

```

注意，`std::shared_ptr` 和 `Rc` 在线程安全性方面略有不同。有关更多详细信息，请参阅类型等效章节。

## 用户定义的构造函数

另一方面，以下示例需要用户定义的复制和移动构造函数，因为它管理一个资源（从 C 库中获取的指针）。在 Rust 中的等效实现需要自定义 `Clone` 特性的实现.^(1)

```rs
#include <cstdlib>
#include <cstring>

// widget.h
struct widget_t;
widget_t *alloc_widget();
void free_widget(widget_t *);
void copy_widget(widget_t *dst, widget_t *src);

// widget.cc
class Widget {
  widget_t *widget;

public:
  Widget() : widget(alloc_widget()) {}

  Widget(const Widget &other) : widget(alloc_widget()) {
    copy_widget(widget, other.widget);
  }

  Widget(Widget &&other) : widget(other.widget) {
    other.widget = nullptr;
  }

  ~Widget() {
    free_widget(widget);
  }
}; 
```

```rs

```

#![allow(unused)] fn main() { mod example { mod widget_ffi {

    // 模拟一个不透明类型。

    // 参考 https://doc.rust-lang.org/nomicon/ffi.html#representing-opaque-structs

    #[repr(C)]

    pub struct CWidget {

        _data: [u8; 0],

        _marker: core::marker::PhantomData<(

            *mut u8,

            core::marker::PhantomPinned,

        )>,

    }

    extern "C" {

        pub fn make_widget() -> *mut CWidget;

        pub fn copy_widget(

            dst: *mut CWidget,

            src: *mut CWidget,

        );

        pub fn free_widget(ptr: *mut CWidget);

    }

}

use self::widget_ffi::*;

struct Widget {

    widget: *mut CWidget,

}

impl Widget {

    fn new() -> Self {

        Widget {

            widget: unsafe { make_widget() },

        }

    }

}

impl Clone for Widget {

    fn clone(&self) -> Self {

        let widget = unsafe { make_widget() };

        unsafe {

            copy_widget(widget, self.widget);

        }

        Widget { widget }

    }

}

impl Drop for Widget {

    fn drop(&mut self) {

        unsafe { free_widget(self.widget) };

    }

}

} }

```rs

```

就像在 C++ 中很少需要为复制和移动构造函数或析构函数编写用户定义的实现一样，在 Rust 中，对于不表示资源的类型，很少需要手动实现 `Clone` 和 `Drop` 特性。

这里有一个例外。如果类型有类型参数，即使克隆应该逐字段进行，也可能会希望手动实现 `Clone`（和 `Copy`）。有关详细信息，请参阅 [标准库中 `Clone` 的文档](https://doc.rust-lang.org/std/clone/trait.Clone.html#how-can-i-implement-clone) 和 [`Copy` 的文档](https://doc.rust-lang.org/std/marker/trait.Copy.html#how-can-i-implement-copy)。

## 可轻易复制的类型

在 C++ 中，当类类型没有任何非平凡的复制构造函数、移动构造函数、复制赋值运算符、移动赋值运算符，并且有一个平凡的析构函数时，它就是可轻易复制的。可轻易复制的类型的值可以通过复制它们的字节进行复制。

在上面的第一个 C++ 示例中，`Age` 是可轻易复制的，但 `Person` 不是。这是因为尽管使用了默认的复制构造函数，但由于 `std::string` 和 `std::shared_ptr` 不是可轻易复制的，所以构造函数不是平凡的。

Rust 使用 `Copy` 特性来指示类型是否可轻易复制。与 C++ 中的可轻易复制的类型一样，Rust 中实现 `Copy` 的类型的值可以通过复制它们的字节进行复制。Rust 要求显式调用 `clone` 方法来复制未实现 `Copy` 的类型的值。

在上面的第一个 Rust 示例中，`Age` 实现了 `Copy` 特性，但 `Person` 没有实现。这是因为 `std::String` 和 `Rc<Person>` 都没有实现 `Copy`。它们没有实现 `Copy` 是因为它们拥有在堆上生存的数据，因此不是可轻易复制的。

Rust 阻止为任何字段不是 `Copy` 的类型实现 `Copy`，但不会阻止为那些不应该逐位复制的类型实现 `Copy`，因为这些类型的意图通常由用户定义的 `Clone` 实现指示。

Rust 不允许为同一类型实现 `Copy` 和 `Drop`。这与 C++ 标准的要求一致，即可轻易复制的类型不应实现用户定义的析构函数。

## 移动构造函数

在 Rust 中，所有类型默认支持移动语义，并且不能（也不需要）定义自定义的移动语义。这是因为 Rust 中的“移动”与 C++ 中的含义不同。在 Rust 中，移动一个值意味着改变拥有该值的所有权。特别是，在移动之后没有“旧”对象需要被销毁，因为编译器会阻止使用已经移动值的变量。

## 赋值运算符

Rust 没有复制或移动赋值运算符。相反，赋值要么通过转移所有权进行移动，要么明确克隆然后移动，或者隐式复制然后移动。

```rs

```

fn main() {

    let x = Box::<u32>::new(5);

    let y = x; // 移动

    let z = y.clone(); // 明确克隆然后移动克隆

    let w = *y; // 隐式复制 Box 的内容然后移动复制

}

```rs

```

对于可能通过用户定义的复制赋值来避免分配的情况，`Clone` 特征有一个额外的名为 `clone_from` 的方法。该方法自动定义，但在实现 `Clone` 特征时可以被覆盖，以提供更有效的实现。默认实现与调用 `Clone::clone` 并执行正常赋值相同。

该方法不用于正常赋值，但在赋值性能很重要且使用更有效的实现可以改进的情况下，可以显式使用。实现可以更高效，因为 `clone_from` 获取要分配值的对象的所有权，因此可以进行诸如重用内存以避免分配等操作。

```rs

```

#![allow(unused)] fn main() { fn go(x: &Vec<u32>) {

    let mut y = vec![0; x.len()];

    // ...

    y.clone_from(&x);

    // ...

}

}

```rs

```

## 性能关注点和 `Copy`

实现 `Copy` 的决定应基于类型的语义，而不是性能。如果复制对象的尺寸是一个关注点，那么应该使用引用（`&T` 或 `&mut T`）或将值放在堆上（`Box<T>` 或 `Rc<T>`）。这些方法对应于通过引用传递，或在 C++ 中使用 `std::unique_ptr` 或 `std::shared_ptr`。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css">

* * *

1.  对于示例的 C++ 版本，另一种常见的方法是使用 `std::unique_ptr` 的 `Deleter` 模板参数。示例中显示的版本是为了使与 Rust 版本的对应关系更清晰而选择的。↩

[点击此处给我们关于此页面的反馈。](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Copy and move constructors)
