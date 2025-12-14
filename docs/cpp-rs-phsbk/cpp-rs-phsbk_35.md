# Lambda、闭包和函数对象

> 原文：[`cel.cs.brown.edu/crp/idioms/closures.html`](https://cel.cs.brown.edu/crp/idioms/closures.html)

## 函数指针

C++ 和 Rust 都允许将函数用作值。在两者中，值都可以具有 [函数指针类型](https://doc.rust-lang.org/std/primitive.fn.html)。

```rs
#include <iostream>

int process(int n) {
  std::cout << n << std::endl;
  return 2 * n;
}

int main() {
  auto f = process;
  // or with type annotation
  // int (*f)(int) = process;

  std::cout << f(42) << std::endl;
} 
```

```rs

fn process(n: i32) -> i32 {

    println!("{}", n);

    2 * n

}

fn main() {

    let f = process;

    // 或使用类型注解

    // let f: fn(i32) -> i32 = process;

    println!("{}", f(42));

}

```

非捕获闭包在 C++ 和 Rust 中也都是可转换的函数指针。在下面的示例中，类型在 C++ 和 Rust 中都可以推断出来，但类型被显式给出，以表明在两种情况下闭包都具有函数指针类型。

```rs
#include <iostream>

int main() {
  int (*f)(int) = [](int n) {
    std::cout << n << std::endl;
    return 2 * n;
  };

  std::cout << f(42) << std::endl;
} 
```

```rs

fn main() {

    let f = |n: i32| {

        println!("{}", n);

        2 * n

    };

    // 或使用类型注解

    // let f: fn(i32) -> i32 = process;

    println!("{}", f(42));

}

```

与 C++ 不同，在 Rust 中可以在其他函数内部定义函数。这具有与在函数外部定义函数相同的意义（即，该函数不是捕获闭包，因此不能捕获外部函数中定义的变量），但函数的名称仅在外部函数内部可用。

```rs

fn main() {

    fn process(n: i32) -> i32 {

        println!("{}", n);

        2 * n

    }

    println!("{}", process(42));

}

```

## Rust 的调用操作符特质

在 C++ 中，任何类都可以实现调用操作符方法 `operator()` 并成为一个函数对象。由 lambda 表达式定义的闭包会自动这样做。在 Rust 中，等效的是调用操作符特质。

| 特质 | 方法 | 描述 |
| --- | --- | --- |
| [`FnOnce<Args>`](https://doc.rust-lang.org/std/ops/trait.FnOnce.html) | `fn call_once(self, args: Args) -> Self::Output` | 至多调用一次 |
| [`FnMut<Args>`](https://doc.rust-lang.org/std/ops/trait.FnMut.html) | `fn call_mut(&mut self, args: Args) -> Self::Output` | 可多次调用且可能修改捕获的变量（类似于 C++ 中的 `mutable` 指令） |
| [`Fn<Args>`](https://doc.rust-lang.org/std/ops/trait.Fn.html) | `fn call(&self, args: Args) -> Self::Output` | 可多次调用且不修改捕获的变量 |

Rust 函数指针实现了所有三个特质。其他闭包根据它们如何使用捕获的变量实现相应的特质。

<details><summary>仅实现 `FnOnce` 的闭包</summary>

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("hi");
  auto f = [x = std::move(x)]() mutable {
    return std::move(x);
  };

  std::cout << f() << std::endl;
  // compiles, but the captured value has been
  // moved
  std::cout << f() << std::endl; // prints ""
} 
```

```rs

fn main() {

let f = {

    let x = String::from("hi");

    // f : FnOnce()

    move || x

};

// f() 等价于 f.call_once()

println!("{}", f()); // 打印 "hi"

// 无法编译--call_once 消耗了 f。

// println!("{}", f());

}

```</details>  <details><summary>实现 `FnMut` 并捕获所有权的闭包</summary>

```rs
#include <string>

int main() {
  std::string x("");
  auto f = [x = std::move(x)]() mutable {
    x.push_back('!');
    return x.size();
  };

  std::cout << f() << std::endl; // prints "1"
  std::cout << f() << std::endl; // prints "2"
} 
```

```rs

fn main() {

    let mut f = {

        let mut x = String::from("");

        // f : FnMut() -> usize

        move || {

            x.push('!');

            x.len()

        }

    };

    println!("{}", f()); // 打印 "1"

    println!("{}", f()); // 打印 "2"

}

```</details>  <details><summary>实现 `FnMut` 并通过可变引用捕获的闭包</summary>

在这种情况下，由于闭包会借用 `x`，因此 `x` 必须在闭包可能被使用的时间内保持有效。因此，`x` 不能像上一个示例中那样在 lambda 的代码块中声明。

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("");
  auto f = [&x]() {
    x.push_back('!');
    return x.size();
  };

  std::cout << f() << std::endl; // prints "1"
  std::cout << f() << std::endl; // prints "2"
} 
```

```rs

fn main() {

    let mut x = String::from("");

    // g : FnMut() -> usize

    let mut f = || {

        x.push('!');

        x.len()

    };

    println!("{}", f()); // 打印 "1"

    println!("{}", f()); // 打印 "2"

}

```</details>  <details><summary>实现 `Fn` 的闭包</summary>

`x` 是否为 `mut` 不影响闭包是否实现 `Fn` 或 `FnMut`。重要的是闭包如何使用 `x`。

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("");
  auto f = [&x]() { return x.size(); };

  std::cout << f() << std::endl; // prints "0"
  std::cout << f() << std::endl; // prints "0"
} 
```

```rs

fn main() {

    let f = {

        let x = String::from("");

        // g : Fn() -> usize

        move || x.len()

    };

    println!("{}", f()); // 打印 "0"

    println!("{}", f()); // 打印 "0"

}

```</details>

## Lambdas and closures

在 C++ 和 Rust 中，捕获闭包的具体类型都不能写出来。在这两种语言中，对于局部变量，这意味着类型必须被推断。

```rs
#include <iostream>
#include <sstream>
#include <string>

int main() {
  std::string greeting = "hello";
  // Can't write the type of the closure
  auto sayHelloTo = & {
    std::ostringstream out;
    out << greeting << " " << who;
    return out.str();
  };

  std::string world("world");
  std::string moon("moon");

  std::cout << sayHelloTo(world) << std::endl;
  std::cout << sayHelloTo(moon) << std::endl;
} 
```

```rs

fn main() {

    let greeting = "hello";

    // 无法写出闭包的类型

    let say_hello_to = |who: &str| {

        format!("{} {}", greeting, who)

    };

    println!("{}", say_hello_to("world"));

    println!("{}", say_hello_to("moon"));

}

```

在 C++ 和 Rust 中，如果闭包是堆分配的，则可以给出类型。在 C++ 中，这是通过 `std::function` 实现的，而在 Rust 中则是通过调用操作符特性实现的。

```rs
#include <functional>
#include <iostream>
#include <sstream>
#include <string>

int main() {
  std::string greeting = "hello";
  // Can't write the type of the closure
  std::function<std::string(std::string &)>
      sayHelloTo(& {
        std::ostringstream out;
        out << greeting << " " << who;
        return out.str();
      });

  std::string world("world");
  std::string moon("moon");

  std::cout << sayHelloTo(world) << std::endl;
  std::cout << sayHelloTo(moon) << std::endl;
} 
```

```rs

fn main() {

    let greeting = "hello";

    // 无法写出闭包的类型

    let say_hello_to: Box<

        dyn Fn(&str) -> String,

    > = Box::new(|who: &str| {

        format!("{} {}", greeting, who)

    });

    println!("{}", say_hello_to("world"));

    println!("{}", say_hello_to("moon"));

}

```

由于 `std::function` 可以是空的，所以上面的示例并不严格等价。然而，由于 `std::function` 通常与值不为空的附加条件一起使用，所以没有 `Option` 包装器的 `Box` 是表示空情况更实用的翻译。

在 Rust 中，也可以用函数操作符特性之一来给出闭包引用的类型。

```rs

fn main() {

    let greeting = "hello";

    // 无法写出闭包的类型

    let say_hello_to = |who: &str| {

        format!("{} {}", greeting, who)

    };

    let say: &dyn Fn(&str) -> String = &say_hello_to;

    println!("{}", say("world"));

    println!("{}", say("moon"));

}

```

此外，在 C++ 和 Rust 中，闭包的返回类型可以作为 lambda 表达式的一部分进行注解。当返回类型无法推断或应该比推断出的类型更具体时，这很有用。在下面的示例中，这是用来以实现接口的值返回，而不是推断出的具体类型。

```rs
#include <iostream>
#include <memory>
#include <string>

// The common interface
struct Debug {
  virtual std::ostream &
  emit(std::ostream &out) const = 0;
};

std::ostream &operator<<(std::ostream &out,
                         const Debug &d) {
  d.emit(out);
  return out;
}

// Two things that implement the interface
struct A : public Debug {
  std::ostream &
  emit(std::ostream &out) const override {
    out << "A";
    return out;
  }
};

struct B : public Debug {
  std::ostream &
  emit(std::ostream &out) const override {
    out << "B";
    return out;
  }
};

int main() {
  // Without the return-type annotation,
  // std::unique_ptr<A> would be inferred.
  auto f = [](std::unique_ptr<A> a,
              std::unique_ptr<B> b)
      -> std::unique_ptr<Debug> { return a; };
  std::cout << *f(std::make_unique<A>(),
                  std::make_unique<B>())
            << std::endl;
} 
```

```rs

// 常见接口

use std::fmt::Debug;

// 实现接口的两个东西

#[derive(Debug)]

struct A;

#[derive(Debug)]

struct B;

fn main() {

    // 没有返回类型注解，

    // Box<A> 将会被推断。

    let f = move |a: Box<A>,

                b: Box<B>|

        -> Box<dyn Debug> { a };

    println!("{:?}", f(Box::new(A), Box::new(B)));

}

```

## 捕获变量

在 C++ 中，捕获指定符用于指示变量是否应该通过引用、复制或移动来捕获。捕获指定符可以一次为所有变量提供，为每个变量提供，或作为默认值与每个变量的特定选择一起提供。

在 Rust 中，变量要么全部通过引用（默认）捕获，要么全部通过移动（使用 `move` 指定符）。为了表达其他捕获策略，需要显式定义引用和副本，并且闭包需要捕获这些变量。

通过显式地创建副本或获取引用的模式表达，利用了 Rust 中块是表达式的这一事实。在需要执行此操作的示例中，注意被分配给持有闭包的变量的块的最后一个语句中没有分号。

以下示例展示了 C++ 中不同捕获变量模式的示例及其在 Rust 中的对应。

<details><summary>通过引用捕获 `x` 和 `y`</summary>

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("hello world");
  std::string y("goodnight moon");

  auto f = [&]() {
    std::cout << x << std::endl;
    std::cout << y << std::endl;
  };

  // x and y borrowed by f, but still available
  std::cout << x << std::endl;
  std::cout << y << std::endl;

  f();
} 
```

```rs

fn main() {

    let x = String::from("hello world");

    let y = String::from("goodnight moon");

    let f = || {

        println!("{}", x);

        println!("{}", y);

    };

    // x 和 y 通过 f 借用，但仍然可用

    println!("{}", x);

    println!("{}", y);

    f();

}

```</details>  <details><summary>通过可变引用捕获 `x` 和 `y`</summary>

C++ 版本与通过可变引用捕获时相同。

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("hello world");
  std::string y("goodnight moon");

  auto f = [&]() {
    x.push_back('!');
    y.push_back('!');
  };

  // x and y borrowed by mutably f, but still
  // available anyway
  std::cout << x << std::endl;
  std::cout << y << std::endl;

  f();

  std::cout << x << std::endl;
  std::cout << y << std::endl;
} 
```

```rs

fn main() {

    let mut x = String::from("hello world");

    let mut y = String::from("goodnight moon");

    // f 需要是可变的，因为它会进行修改

    // 它捕获的变量

    let mut f = || {

        x.push('!');

        y.push('!');

    };

    // x 和 y 通过 f 可变借用，因此

    // 不能在这里使用

    // println!("{}", x);

    println!("{}", y);

    f();

    println!("{}", x);

    println!("{}", y);

}

```</details>  <details><summary>通过值复制 `x` 和 `y` 来捕获</summary>

在 C++ 中，这要求 lambda 有 `mutable` 指定符。在 Rust 中，这要求

+   为闭包捕获的值创建副本之前发生，

+   将这些副本标记为可变的 `mut`，

+   将闭包本身标记为可变的 `mut`，并且

+   使用 `move` 指定符将副本的所有权移动到闭包中。

表示它们可以通过实现 `Copy` 特性进行简单复制的类型（构造函数/复制和移动构造函数）不需要显式克隆。

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("hello world");
  std::string y("goodnight moon");

  auto f = [=]() mutable {
    x.push_back('!');
    y.push_back('!');
  };

  // copies of x and y owned by f, originals
  // still available
  std::cout << x << std::endl;
  std::cout << y << std::endl;

  f();

  // still don't have the !, since the copies
  // were modified not the originals
  std::cout << x << std::endl;
  std::cout << y << std::endl;
} 
```

```rs

fn main() {

    let x = String::from("hello world");

    let y = String::from("goodnight moon");

    let mut f = {

        // 使用副本来遮蔽外部变量。

        // 这需要在

        // 闭包表达式。

        let mut x = x.clone();

        let mut y = y.clone();

        move || {

            x.push('!');

            y.push('!');

        }

    };

    // f 拥有 x 和 y 的副本，原始值

    // 仍然可用

    println!("{}", x);

    println!("{}", y);

    f();

    // 仍然没有 !，因为副本

    }

    println!("{}", x);

    println!("{}", y);

}

```</details>  <details><summary>将 `x` 和 `y` 通过值捕获</summary>

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("hello world");
  std::string y("goodnight moon");

  auto f = [x = std::move(x),
            y = std::move(y)]() {
    std::cout << x << std::endl;
    std::cout << y << std::endl;
  };

  // x and y moved into f,
  // empty strings left behind.
  std::cout << x << std::endl;
  std::cout << y << std::endl;

  f();
} 
```

```rs

fn main() {

    let x = String::from("hello world");

    let y = String::from("goodnight moon");

    msg: String,

    }

        println!("{}", x);

        println!("{}", y);

    };

    // x 和 y 被移动到 f 中，

    // 原始变量不能使用

    // println!("{}", x);

    // println!("{}", y);

    let f = my_closure.as_fn();

}

```</details>  <details><summary>将 `x` 通过值捕获，捕获 `y` 为引用</summary>

```rs
#include <iostream>
#include <string>

int main() {
  std::string x("hello world");
  std::string y("goodnight moon");

  auto f = [x = std::move(x), &y]() mutable {
    x.push_back('!');
    std::cout << x << std::endl;
    std::cout << y << std::endl;
  };

  // x moved into f, y borrowed by f
  std::cout << x << std::endl;
  std::cout << y << std::endl;

  f();
} 
```

```rs

fn main() {

    let mut x = String::from("hello world");

    let y = String::from("goodnight moon");

    let mut f = {

        let y = &y;

        // 实际上通过值捕获了 x 和 y，

        // 值，但 y 是引用

        move || {

            x.push('!');

            // 被修改的是副本，不是原始值

            println!("{}", y);

        }

    };

    // x 被移动到 f 中，y 被 f 借用

    // println!("{}", x);

    println!("{}", y);

    f();

}

```</details>

## 函数对象

与 C++ 不同，在 Rust 中只有函数和闭包实现了函数调用操作符特质。直接实现这些特质的可能性 [目前还不是稳定 Rust 的一部分](https://github.com/rust-lang/rust/issues/29625)。

相反，可以实现一个 转换函数。然而，由于 `impl Trait` 语法不能用于特质实现，因此不能为这个目的实现标准转换特质 `From` 和 `Into`。因此，必须定义一个单独的方法。^(1) 而不是使用 `impl Trait` 语法。

```rs
#include <cstddef>
#include <iostream>
#include <string>

struct MyClosure {
  std::string msg;

  std::size_t operator()() {
    std::cout << msg << std::endl;
    return msg.size();
  }
};

int main() {
  MyClosure myClosure{"hello world"};
  myClosure();
} 
```

```rs

struct MyClosure {

    f();

}

impl MyClosure {

    fn as_fn(&self) -> impl Fn() -> usize {

        move || {

            println!("{}", self.msg);

            self.msg.len()

        }

    println!("{}", x);

}

fn main() {

    let my_closure = MyClosure {

        msg: String::from("hello world"),

    };

    let f = move || {

    f();

}

```

在 Rust 2024 年之前的版本中，上述示例需要使用精确捕获注解 `use<'a>` 语法（[`doc.rust-lang.org/std/keyword.use.html#precise-capturing`](https://doc.rust-lang.org/std/keyword.use.html#precise-capturing)）来指定返回的闭包从参数中借用，否则无法推断出生命周期限制。

## 成员函数作为函数指针

在 C++ 中，成员函数指针可以使用 `.*` 操作符调用，或者使用 `std::mem_fn` 转换为 `std::function` 值，从而可以像其他 `std::function` 值一样使用。当在派生类上调用时，调用的是取地址的方法还是派生类中重写的方法取决于该方法是否定义为虚函数。

在 Rust 中，成员函数的指针是普通函数指针。例如，具有 `&self` 参数的类型 `T` 的方法是一个其第一个参数类型为 `&T` 的函数。当方法通过特例命名时，函数的第一个参数类型为 `&dyn T`，并具有生命周期限制。确定是否在成员函数指针的使用中涉及 vtable 是在引用方法时确定的，而不是在定义方法时。

```rs
#include <cassert>
#include <functional>
#include <iostream>

struct Interface {
  virtual void showVirtual() = 0;
};

struct A : public Interface {
  void show() {
    std::cout << "A" << std::endl;
  }

  void showVirtual() override {
    std::cout << "A" << std::endl;
  }
};

struct B : public Interface {
  void showVirtual() override {
    std::cout << "B" << std::endl;
  }

  void show() {
    std::cout << "B" << std::endl;
  }
};

int main() {
  auto showV = &Interface::showVirtual;
  auto showA = &A::show;
  auto showB = &B::show;

  A a;
  B b;

  (a.*showV)(); // prints A
  (b.*showV)(); // prints B

  (a.*showA)(); // prints A
  (b.*showB)(); // prints B
} 
```

```rs

trait Interface {

    fn show(&self);

}

struct A;

impl Interface for A {

    fn show(&self) {

        println!("A");

    }

}

struct B;

impl Interface for B {

    fn show(&self) {

        println!("B");

    }

}

fn main() {

    // 类型可以推断，但给出以示说明

    // 它们只是函数指针

    let show_a: fn(&A) = A::show;

    let show_b: fn(&B) = B::show;

    let show_v: fn(&(dyn Interface + 'static)) =

        Interface::show;

    show_a(&A); // 打印 A

    show_b(&B); // 打印 B

    show_v(&A); // 打印 A

    show_v(&B); // 打印 B

}

```

## 闭包作为参数

在 C++ 和 Rust 中，也可以接受未装箱的闭包作为参数。就像在 C++ 中使用 `auto` 作为参数类型使函数实际上成为一个函数模板一样，在 Rust 中使用 `impl Trait` 作为参数类型使函数泛型。生成的泛型函数是静态检查的，就像显式给出类型参数和边界时一样。

```rs
#include <iostream>

int apply_to_0(auto f) {
  return f(0);
}

int main() {
  int x = 1;
  auto f(= { return n + x; });
  std::cout << apply_to_0(f) << std::endl;
} 
```

```rs

fn apply_to_0(f: impl FnOnce(i32) -> i32) -> i32 {

    f(0)

}

fn main() {

    let x = 1;

    let f = move |n: i32| x + n;

    println!("{}", apply_to_0(&f));

}

```

在 Rust 中接受闭包作为类型参数时，最好指定所需的最不限制接口，以确定闭包将被如何使用。

使用 `FnOnce` 作为边界是最不限制的，因此应该使用它，以便接受闭包作为参数的函数尽可能与尽可能多的闭包兼容。`FnOnce` 与 `Fn` 和 `FnMut` 一起工作，因为为 `&Fn` 和 `&FnMut` 提供了 `FnOnce` 特例实现。`FnMut` 特例是下一个最限制的，然后是 `Fn`，最后是实际函数指针，其类型以小写 `fn` 编写。

在 C++ 和 Rust 中，也可以传递闭包的引用或指针。在以下示例中，闭包在 C++ 和 Rust 中都在动态分配的存储中。

```rs
#include <functional>
#include <iostream>

int apply_to_0(std::function<int(int)> f) {
  return f(0);
}

int main() {
  int x = 1;
  // closure is on heap
  auto f(std::function(
      = { return n + x; }));
  std::cout << apply_to_0(f) << std::endl;
} 
```

```rs

fn apply_to_0(f: Box<dyn FnOnce(i32) -> i32>) -> i32 {

    f(0)

}

fn main() {

    let x = 1;

    let f = Box::new(move |n: i32| x + n);

    println!("{}", apply_to_0(f));

}

```

`FnOnce` 可以在 `Box` 中调用，因为盒子拥有特例对象的所有权，但不在引用中调用，因为引用没有所有权。`Fn` 和 `FnMut` 没有相同的限制。

## 返回闭包

在 C++中，可以使用`auto`或`decltype(auto)`作为返回闭包的函数的返回类型。在 Rust 中，再次使用`impl Trait`语法。就像在 C++中使用`auto`这种方式并不表示简化的函数模板一样，它也不表示泛型函数。相反，类型是推断的，并且必须满足特质的约束。

```rs
#include <iostream>

decltype(auto) makeConst(int n) {
  return [n]() { return n; };
}

int main() {
  auto f = makeConst(42);
  std::cout << f() << std::endl;
} 
```

```rs

fn make_const(n: i32) -> impl Fn() -> i32 {

    move || n

}

fn main() {

    let f = make_const(42);

    println!("{}", f());

}

```

在 C++中使用`decltype`来命名闭包的地方，例如在模板类中返回闭包时，在 Rust 中则使用`impl Trait`语法。如果需要在 let 绑定中指定类型，可以使用下划线`_`来表示类型中闭包的类型部分应该被推断。

```rs

struct Wrapper<T>(T);

fn make_closure() -> Wrapper<impl Fn(i32) -> i32>

{

    let x = 1;

    Wrapper(move |n: i32| x + n)

}

fn main() {

    let w: Wrapper<_> = make_closure();

    w.0(0);

}

```

有几个其他地方`decltype`可以工作，但`impl Trait`还不能，例如[`Fn`特质的输出类型](https://github.com/rust-lang/rust/issues/99697)。这意味着在 Rust 中可以定义返回闭包的闭包，但不能为它们指定类型，因此不能从函数中返回它们。下面的代码在 C++中可以编译，但在 Rust 中由于这个原因无法编译。

```rs
decltype(auto) makeClosure(int n) {
  return [n]() { return [n]() { return n; }; };
} 
```

```rs
// Does not compile: not yet supported
fn make_closure(
    n: i32,
) -> impl Fn() -> impl Fn() -> i32 {
    move || move || n
}
```

## 模板 lambda

Rust 不支持泛型闭包。因此，以下在 Rust 中没有等效功能。

```rs
#include <string>

int main() {
  int n = 0;

  auto idCounter = [&]<typename T>(T x) {
    n++;
    return x;
  };

  int y = idCounter(5);
  std::string z =
      idCounter.template operator()<std::string>(std::string("hi"));
} 
```

然而，如果 lambda 没有捕获任何内容，可以通过使用内部函数定义在 Rust 中写出等效的代码。

```rs
#include <string>

int main() {
  auto id = []<typename T>(T x) { return x; };
  int y = id(5);
  std::string z = id(std::string("hi"));
} 
```

```rs

fn main() {

    fn id<T>(x: T) -> T {

        x

    }

    id(5);

    id(String::from("hi"));

}

```

## 部分应用和`std::bind`

Rust 标准库中没有与 C++模板`std::bind`等效的功能。在 Rust 中表达部分应用的传统方式是写出 lambda 表达式。

```rs
#include <cassert>
#include <functional>

int add(int x, int y) {
  return x + y;
}

int main() {
  using namespace std::placeholders;

  auto addTen = std::bind(add, 10, _1);
  assert(42 == addTen(32));
} 
```

```rs

fn add(x: i32, y: i32) -> i32 {

    x + y

}

fn main() {

    let add_ten = move |y| add(10, y);

    assert_eq!(42, add_ten(32));

}

```

第三方 crate [partial_application](https://docs.rs/partial_application/latest/partial_application/) 提供了类似于`std::bind`的功能，使用 Rust 宏实现。

```rs
use partial_application::*;

fn add(x: i32, y: i32) -> i32 {
    x + y
}

fn main() {
    let add_ten = partial!(move add => 10, _);

    assert_eq!(42, add_ten(32));
}
```

## 返回捕获变量的引用

在 Rust 中，闭包无法返回对捕获变量的引用。这是由于`Fn`族特质的定义限制造成的：`Fn::Output`没有表达生命周期边界的方法。

```rs
#include <iostream>
#include <string>

int main() {
  std::string msg("hello world");
  auto f = [=]() -> const std::string & {
    return msg;
  };
  std::cout << f() << std::endl;
} 
```

```rs
fn main() {
    let msg = String::from("hello world");
    // fails to compile!
    let f = move || &msg;

    println!("{}", f());
}
```

在 Rust 中解决这种限制的方法包括：要么堆分配并使用共享指针 `Rc`，要么定义一个新的特质而不是使用 `Fn` 特质之一。以下示例展示了使用 [泛型关联类型](https://doc.rust-lang.org/reference/items/associated-items.html#r-items.associated.type.generic) 定义通用 `Fn` 特质的特质。然而，在实践中，通常更好的做法是为每个用例定义一个自定义特质，或者如果单个结构体就足够了，则完全省略特质。

```rs

trait Closure<Args> {

    // 生命周期参数允许表达

    // 界限。

    类型 Output<'a>

    where

        Self: 'a;

    // 来自 self 的界限可以是

    // 提供给 Output。

    fn call<'a>(

        &'a self,

        args: Args,

    ) -> Self::Output<'a>;

}

struct MyClosure {

    msg: String,

}

实现闭包类型 `Closure<()>` for MyClosure {

    类型 Output<'a> = &'a str;

    fn call(&self, _: ()) -> &str {

        &self.msg

    }

}

fn main() {

    let f = MyClosure {

        msg: String::from("hello world"),

    };

    println!("{}", f.call(()));

}

```

## 闭包、所有权和 `FnOnce`

在 Rust 中，闭包是其中借用检查可能会让 C++ 程序员感到沮丧的部分。这通常不是因为生命周期，在 C++ 中也需要类似地考虑生命周期，而是因为 C++ 默认使用复制语义，而 Rust 默认使用移动语义。例如，对早期示例中的一次小调整无法编译。

```rs
fn main() {
    let greeting = "hello ".to_string();

    // Can't write the type of the closure
    let say_hello_to = move |who: &str| {
        greeting + who
    };

    println!("{}", say_hello_to("world"));
    println!("{}", say_hello_to("moon"));
}
```

这无法编译，因为 `+` 运算符会获取 `greeting` 的所有权，这使得它对于后续调用不再可访问。因此，闭包只实现了 `FnOnce`，而不是 `Fn`，因此只能调用一次，因为调用会获取闭包本身的所有权。

```rs
error[E0382]: use of moved value: `say_hello_to`
 --> example.rs:9:20
  |
8 |     println!("{}", say_hello_to("world"));
  |                    --------------------- `say_hello_to` moved due to this call
9 |     println!("{}", say_hello_to("moon"));
  |                    ^^^^^^^^^^^^ value used here after move
  |
note: closure cannot be invoked more than once because it moves the variable `greeting` out of its environment
 --> example.rs:6:26
  |
6 |         move |who: &str| greeting + who;
  |                          ^^^^^^^^
note: this value implements `FnOnce`, which causes it to be moved when called
 --> example.rs:8:20
  |
8 |     println!("{}", say_hello_to("world"));
  | 
```

In many cases like this, the answer is to clone the value so that the copy owned by the closure can be retained for future invocations.

```rs

fn main() {

    let greeting = "hello ".to_string();

    // 无法写出闭包的类型

    let say_hello_to = move |who: &str| {

        greeting.clone() + who

    };

    println!("{}", say_hello_to("world"));

    println!("{}", say_hello_to("moon"));

}

```

如果克隆是不希望的，因为它太昂贵，那么闭包需要重新设计以避免放弃其捕获变量的所有权。

## 文档最佳实践

C++ 常常建议在 lambda 表达式中显式列出捕获项，尤其是在闭包将超出其上下文的情况。这样做是为了帮助推理捕获项的生命周期，以确保闭包不会超出它捕获的任何对象。

在 Rust 中，关于捕获的生命周期的决策必须与 C++ 中做出相同的决策，但编译器会跟踪它们，而不是需要手动推理。也就是说，尽管闭包的类型无法表达，但它仍然包括通过引用捕获的变量的生命周期，因此会像任何其他结构一样进行检查。

这导致在 Rust 中记录闭包的最佳实践不包括列举捕获，即使在 C++ 中可能会这样做的情况下。

闭包中捕获内容的可破坏性也是如此。在 上一节 中涉及 `FnOnce` 函数的例子可能最初是一个令人沮丧的点，但该行为的好处是减少了文档和推理的负担。

<link rel="stylesheet" type="text/css" href="../quiz/style.css">

* * *

1.  也就是说，既不能编写 `impl From<&MyClosure> for impl Fn() -> usize {...}` 也不能编写 `impl Into<impl Fn() -> usize> for &MyClosure {...}`。 ↩

[点击此处给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Lambdas, closures, and function objects)
