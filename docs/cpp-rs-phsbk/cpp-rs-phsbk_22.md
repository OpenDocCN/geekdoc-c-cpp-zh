# 匿名命名空间和静态

> 原文：[`cel.cs.brown.edu/crp/idioms/encapsulation/anonymous_namespaces.html`](https://cel.cs.brown.edu/crp/idioms/encapsulation/anonymous_namespaces.html)

C++ 中的匿名命名空间用于避免不同翻译单元之间的符号冲突。这种冲突违反了[单一定义规则](https://timsong-cpp.github.io/cppwp/n4950/basic.def.odr#14)，并导致未定义的行为（在最坏的情况下表现为链接错误）。

例如，如果不使用匿名命名空间，以下代码将导致未定义的行为（由于使用 `inline` 在对象文件中产生弱符号，因此没有链接错误）。

```rs
/// a.cc
namespace {
    inline void common_function_name() {
        // ...
    }
}

/// b.cc
namespace {
    inline void common_function_name() {
        // ...
    }
} 
```

C++ 静态声明也用于通过使声明具有内部链接（因此不在翻译单元外部可见）来实现相同的目标。

Rust 通过同时控制链接和可见性来避免链接问题，声明总是也是定义。与翻译单元不同，程序以模块的形式组织，它们提供命名空间和定义的可见性控制，使 Rust 编译器能够保证不会发生符号冲突问题。

以下 Rust 程序在避免两个函数冲突的同时，使它们在定义文件内可用，与上面的 C++ 程序达到相同的目标。

```rs

```

#![allow(unused)] fn main() { // a.rs

mod a { fn common_function_name() {

    // ...

}

}

// b.rs

mod b { fn common_function_name() {

    // ...

}

} }

```rs

```

此外，

1.  与 C++ 命名空间不同，Rust 模块（它们提供命名空间以及可见性控制）只能定义一次，并且这是由编译器检查的。

1.  每个文件[定义了一个模块，该模块必须显式包含在模块层次结构中](https://doc.rust-lang.org/stable/book/ch07-05-separating-modules-into-different-files.html)。

1.  来自 Rust 包（库）的模块始终带有某个根模块名称，因此它们不会冲突。如果它们会冲突，[根模块名称必须替换为用户选择的名称](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html#renaming-dependencies-in-cargotoml)。

## 关于 C 兼容性的注意事项

当使用由 Rust 不管理的库时，如果对象文件中存在符号冲突，可能会出现通常的问题。这在使用 C 或 C++ 静态或动态库时可能会发生。当使用为在 C 或 C++ 程序中使用而构建的 Rust 静态或动态库时，也可能发生。

Rust 提供了 `#[unsafe(no_mangle)]` 来绕过名称修饰，以便产生可以从 C 或 C++ 中轻松引用的函数。这也可能导致由于名称冲突而导致的未定义行为。

<link rel="stylesheet" type="text/css" href="../../quiz/style.css"> [点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Anonymous%20namespaces%20and%20static)
