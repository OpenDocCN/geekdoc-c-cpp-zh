# 头文件

> [`cel.cs.brown.edu/crp/idioms/encapsulation/headers.html`](https://cel.cs.brown.edu/crp/idioms/encapsulation/headers.html)

C++中头文件的一个用途是将定义在一个翻译单元中的声明暴露给其他翻译单元，而无需在多个文件中重复声明。按照惯例，未包含在头文件中的声明被认为是定义翻译单元的私有声明（尽管，为了强制执行此惯例，需要其他机制，例如匿名命名空间）。

相比之下，Rust 既不使用文本包含的头文件，也不使用前向声明。相反，Rust 模块同时控制可见性和链接，并为其他模块提供公开定义。

```rs
// person.h
#include <string>

class Person {
  std::string name;

public:
  Person(std::string name) : name(name) {}
  const std::string &getName();
};

// person.cc
#include "person.h"

const std::string &Person::getName() {
  return this->name;
}

// client.cc
#include <string>
#include "person.h"

int main() {
  Person p("Alice");
  const std::string &name = p.getName();

  // ...
} 
```

```rs
// person.rs
pub struct Person {
    name: String,
}

impl Person {
    pub fn new(name: String) -> Person {
        Person { name }
    }

    pub fn name(&self) -> &String {
        &self.name
    }
}

// client.rs
mod person;

use person::*;

fn main() {
    let p = Person::new("Alice".to_string());
    // doesn't compile, private field
    // let name = p.name;
    let name = p.name();

    //...
}
```

在`person.rs`中，`Person`类型是公开的，但`name`字段不是。这防止了直接构造该类型的值（类似于私有成员在 C++中防止聚合初始化）以及防止字段访问。通过`pub`可见性声明，静态方法`Person::new(String)`和方法`Person::name()`被暴露给模块的客户。

在`client`模块中，`mod`声明将`person.rs`的内容定义为名为`person`的子模块。`use`声明将`person`模块的内容引入作用域。

## 差异的本质

C++程序是一系列翻译单元的集合。头文件是必需的，以便使提供其他翻译单元的定义的前向声明变得可管理。

Rust 程序是一个模块树。一个模块中的定义可以基于模块自身定义中给出的可见性声明访问其他模块中的项。

## 子模块和附加可见性功能

模块和可见性声明比上述示例中显示的更强大。关于如何使用模块、`pub`和`use`来实现封装目标的更多细节，请参阅私有成员和友元章节。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Header files)
