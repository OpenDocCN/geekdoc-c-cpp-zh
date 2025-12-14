# 移动成员

> 原文：[`cel.cs.brown.edu/crp/idioms/null/moved_members.html`](https://cel.cs.brown.edu/crp/idioms/null/moved_members.html)

在 Rust 中，将值从变量或字段中移出比 C++ 中更为明确。一个可能被移出而留下空白的值在 Rust 中需要使用 `Option<Box<T>>` 类型来表示，而在 C++ 中则只需一个 `std::unique_ptr<T>`。

```rs
#include <memory>

void readMailbox(std::unique_ptr<int> &mailbox,
                 std::mutex mailboxMutex) {
  std::lock_guard<std::mutex> guard(mailboxMutex);

  if (!mailbox) {
    return;
  }
  int x = *mailbox;
  mailbox = nullptr;
  // use x
} 
```

```rs

```

#![allow(unused)] fn main() { use std::sync::Arc;

use std::sync::Mutex;

fn read(mailbox: Arc<Mutex<Option<i32>>>) {

    let Ok(mut x) = mailbox.lock() else {

        return;

    };

    let x = x.take();

    // 使用 x

}

}

```rs

```

此外，当从可变引用内部获取值的所有权时，必须在原位置留下某些内容。这可以通过使用 `std::mem::swap` 来完成，许多类似容器的类型都有使常见所有权交换更方便的方法，例如在先前的示例中看到的 `Option::take`（[Option::take](https://doc.rust-lang.org/std/option/enum.Option.html#method.take)），还有 `Option::replace` 或 `Vec::swap`（[Vec::swap](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.swap_remove)）。

## 删除已移动对象

现代 C++ 中 null 指针的另一个常见用途是作为已移动对象的成员值，以便析构函数仍然可以安全地被调用。例如，

```rs
#include <cstdlib> #include <cstring>   // widget.h
struct widget_t;
widget_t *alloc_widget();
void free_widget(widget_t*);
void copy_widget(widget_t* dst, widget_t* src);

// widget.cc
class Widget {
    widget_t* widget;
public:
 Widget() : widget(alloc_widget()) {}   Widget(const Widget &other) : widget(alloc_widget()) { copy_widget(widget, other.widget); }      Widget(Widget &&other) : widget(other.widget) {
        other.widget = nullptr;
    }

    ~Widget() {
        free_widget(widget);
    }
}; 
```

Rust 中对移动对象的观念不涉及留下一个将在其上调用析构函数的对象，因此这种对 null 的使用没有对应的惯用语。有关更多详细信息，请参阅复制和移动构造函数章节。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Moved%20members)
