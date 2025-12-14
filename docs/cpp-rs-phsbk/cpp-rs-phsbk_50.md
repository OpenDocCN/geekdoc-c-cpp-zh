# 库

> 原文：[`cel.cs.brown.edu/crp/etc/libraries.html`](https://cel.cs.brown.edu/crp/etc/libraries.html)

C++程序倾向于使用操作系统发行版中提供的库，或者使用供应商提供的库。

Rust 程序通常依赖于一个名为[crates.io](https://crates.io/)的中心 Rust 库注册表（称为"crates"），以及一个由这些 crates 的代码注释生成的中心文档仓库[docs.rs](https://docs.rs/)。crates 的依赖关系通过[Cargo 包管理器](https://doc.rust-lang.org/cargo/index.html)进行管理。

[Lib.rs](https://lib.rs/)是一个按类别组织流行 crates 的好资源。

## 一些特定的替代方案

| C++库 | Rust 替代方案 |
| --- | --- |
| STL UTF-16 和 UTF-32 字符串 | [widestring](https://docs.rs/widestring/latest/widestring/) |
| STL 随机数 | [rand](https://github.com/rust-random/rand) |
| STL 正则表达式 | [regex](https://github.com/rust-lang/regex) |
| 反射 | [bevy_reflect](https://docs.rs/bevy_reflect/latest/bevy_reflect/) |
| Boost.Test | [cargo test](https://doc.rust-lang.org/book/ch11-01-writing-tests.html) |
| pybind11 | [PyO3](https://pyo3.rs/) |
| OpenSSL | [rustls](https://github.com/rustls/rustls) |

如果您使用了一个无法找到 Rust 替代方案的 C++库，请通过以下链接提供反馈，让我们知道库的名称和用途。

## 供应链管理

在管理库供应链很重要的情况下，Cargo 可以使用[自定义自管理或组织管理的注册表](https://doc.rust-lang.org/cargo/reference/registries.html)或使用从 crates.io 获取的依赖关系的供应商版本[cargo vendor](https://doc.rust-lang.org/cargo/commands/cargo-vendor.html)。

这两种方法都提供了作为供应链安全一部分审查依赖关系的机制。

不涉及供应商或自定义注册表的供应链安全解决方案正在[进行中](https://github.com/rust-lang/rfcs/pull/3724)。

[点击此处为此页面提供反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=Libraries)
