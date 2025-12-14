# 枚举

> 原文：[`cel.cs.brown.edu/crp/idioms/data_modeling/enums.html`](https://cel.cs.brown.edu/crp/idioms/data_modeling/enums.html)

在 C++ 中，枚举通常用于模拟一组固定的替代方案，特别是当每个枚举符对应于一个特定的整数值时，例如在处理硬件、系统调用或协议实现时。

例如，GPIO 引脚的各种模式可以模拟为枚举，这将限制使用该模式的方法只能使用有效值。

虽然 Rust 枚举更通用，但它们仍然可以用于这种类型的建模。

```rs
#include <cstdint>

enum Pin : uint8_t {
  Pin1 = 0x01,
  Pin2 = 0x02,
  Pin3 = 0x04
};

enum Mode : uint8_t {
  Output = 0x03,
  Pullup = 0x04,
  Analog = 0x27
  // ...
};

void low_level_set_pin(uint8_t pin, uint8_t mode);

void set_pin_mode(Pin pin, Mode mode) {
  low_level_set_pin(pin, mode);
} 
```

```rs

#![allow(unused)] fn main() { #[repr(u8)]

#[derive(Clone, Copy)]

enum Pin {

    Pin1 = 0x01,

    Pin2 = 0x02,

    Pin3 = 0x04,

}

#[repr(u8)]

#[derive(Clone, Copy)]

enum Mode {

    Output = 0x03,

    Pullup = 0x04,

    Analog = 0x27,

    // ...

}

extern "C" {

    fn low_level_set_pin(pin: u8, mode: u8);

}

fn set_pin_mode(pin: Pin, mode: Mode) {

    unsafe {

        low_level_set_pin(pin as u8, mode as u8)

    };

}

}

```

`#[repr(u8)]` 属性确保枚举的表示与字节相同（就像在 C++ 中声明枚举的底层类型一样）。然后可以使用 `as` 将枚举值自由转换为底层类型。

在 C++ 中，从整数转换为枚举的标准方式是静态转换。然而，这[要求用户自己检查转换的有效性](https://eel.is/c++draft/expr.static.cast#9)。通常，转换是通过一个函数完成的，该函数检查要转换的值是否是有效的枚举值。

在 Rust 中，执行转换的标准方式是为类型实现 `TryFrom` 特性，然后使用 `try_from` 方法或 `try_into` 方法。

```rs
#include <cstdint>   enum Pin : uint8_t {
 Pin1 = 0x01, Pin2 = 0x02, Pin3 = 0x04 };   struct InvalidPin {
    uint8_t pin;
};

Pin to_pin(uint8_t pin) {
  // The values are not contiguous, so we can't
  // just check the bounds and then cast.
  switch (pin) {
  case 0x1: { return Pin1; }
  case 0x2: { return Pin2; }
  case 0x4: { return Pin3; }
  }
  throw InvalidPin{pin};
}

int main() {
  try {
    Pin p(to_pin(2));
  } catch (InvalidPin &e) {
    return 0;
  }

  // use pin p
} 
```

```rs

#[repr(u8)] #[derive(Clone, Copy)] enum Pin {

Pin1 = 0x01, Pin2 = 0x02, Pin3 = 0x04, }   use std::convert::TryFrom;

struct InvalidPin(u8);

impl TryFrom<u8> for Pin {

    type Error = InvalidPin;

    fn try_from(

        value: u8,

    ) -> Result<Self, Self::Error> {

        match value {

            0x01 => Ok(Pin::Pin1),

            0x02 => Ok(Pin::Pin2),

            0x04 => Ok(Pin::Pin3),

            pin => Err(InvalidPin(pin)),

        }

    }

}

fn main() {

let Ok(p) = Pin::try_from(2) else {

    return;

};

// use pin p

}

```

查看异常和错误处理以获取如何优雅地处理 `try_from` 结果的示例。

如果低级性能比内存安全更重要，`std::mem::transmute` 类似于 C++ 的 reinterpret_cast，但需要不安全 Rust，因为其使用可能导致未定义行为。除非接口可以实际保证调用永远不会使用无效值，否则不应将 `std::mem::transmute` 的使用隐藏在可以从安全 Rust 调用的接口后面。

## 枚举和方法

在 C++ 中，枚举不能有方法。相反，为了模拟具有方法的枚举，必须为枚举定义一个包装类，并在该包装类上定义方法。在 Rust 中，可以在具有 `impl` 块的枚举上定义方法，就像任何其他类型一样。

```rs
#include <cstdint>

// Actual enum
enum PinImpl : uint8_t {
  Pin1 = 0x01,
  Pin2 = 0x02,
  Pin3 = 0x04
};

class LastPin{};

// Wrapper type
struct Pin {
  PinImpl pin;

  // Conversion constructor so that PinImpl can be
  // used as a Pin.
  Pin(PinImpl p) : pin(p) {}

  // Conversion method so wrapper type can be
  // used with switch statement.
  operator PinImpl() {
    return this->pin;
  }

  Pin next() const {
    switch (pin) {
    case Pin1:
      return Pin(Pin2);
    case Pin2:
      return Pin(Pin3);
    default:
      throw LastPin{};
    }
  }
}; 
```

```rs

#![allow(unused)] fn main() { #[repr(u8)]

#[derive(Clone, Copy)]

enum Pin {

    Pin1 = 0x01,

    Pin2 = 0x02,

    Pin3 = 0x04,

}

struct LastPin;

impl Pin {

    fn next(&self) -> Result<Self, LastPin> {

        match self {

            Pin::Pin1 => Ok(Pin::Pin2),

            Pin::Pin2 => Ok(Pin::Pin3),

            Pin::Pin3 => Err(LastPin),

        }

    }

}

}

```

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Enums)
