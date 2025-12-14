# 继承和实现重用

> 原文：[`cel.cs.brown.edu/crp/idioms/data_modeling/inheritance_and_reuse.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/inheritance_and_reuse.html)

Rust 没有继承，因此 Rust 中重用实现的主要方式是组合、聚合和泛型。

然而，Rust 特性确实支持默认方法，这类似于使用继承来重用实现的一个简单案例。例如，在以下示例中，使用了两个虚拟方法来支持由抽象类提供的实现方法。

```rs
#include <iostream>
#include <string>

class Device {
public:
    virtual void powerOn() = 0;
    virtual void powerOff() = 0;

    virtual void resetDevice() {
        std::cout << "Resetting device..." << std::endl;
        powerOff();
        powerOn();
    }

    virtual ~Device() {}
};

class Printer : public Device {
    bool powered = false;
public:
    void powerOn() override {
        this.powered = true;
        std::cout << "Printer is powered on." << std::endl;
    }

    void powerOff() override {
        this.powered = false;
        std::cout << "Printer is powered off." << std::endl;
    }
};

int main() {
    Printer myPrinter;
    myPrinter.resetDevice();
} 
```

```rs

trait Device {

    fn power_on(&mut self);

    fn power_off(&mut self);

    fn reset_device(&mut self) {

        println!("Resetting device...");

        self.power_off();

        self.power_on();

    }

}

struct Printer {

    powered: bool,

}

impl Printer {

    fn new() -> Printer {

        Printer { powered: false }

    }

}

impl Device for Printer {

    fn power_on(&mut self) {

        self.powered = true;

        println!("Printer is powered on");

    }

    fn power_off(&mut self) {

        self.powered = false;

        println!("Printer is powered off");

    }

}

fn main() {

    let mut p = Printer::new();

    p.reset_device();

}

```

在实践中，如果预期不会覆盖 `resetDevice()` 方法，C++ 中的 `Device` 类的 `resetDevice()` 方法可能被设置为非虚拟。为了使其与 Rust 示例保持一致，我们在这里将其设置为虚拟，因为 Rust 特性既可以用于动态分派也可以用于静态分派（在静态分派的情况下没有 vtable 开销）。

Rust 特性与抽象类在几个方面有所不同。例如，Rust 特性不能定义数据成员，也不能定义私有或受保护的成员方法。这限制了使用特性实现模板方法模式的有效性。

Rust 特性也不能私有实现。在任何既可以看到特性又可以看到实现该特性的类型的地方，特性的方法都会作为类型的方法可见。

特性可以相互继承，包括多重继承。然而，在现代 C++ 中，Rust 的继承层次通常较浅。但在具有复杂多重继承的情况下，Rust 中不会出现菱形问题，因为特性不能覆盖其他特性的实现。因此，通往公共父特性所有路径都会解析到相同的实现。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Inheritance and implementation reuse)
