# 构建系统（例如，CMake）

> [`cel.cs.brown.edu/crp/etc/build_systems.html`](https://cel.cs.brown.edu/crp/etc/build_systems.html)

C++ 和 Rust 生态系统之间一个主要的不同之处在于，C++ 和 C 库通常由操作系统发行版提供，或者包含在项目的仓库中，而 Rust 有一个中心化的语言特定包注册中心，称为 [crates.io](https://crates.io/)。

由于 Rust 的构建工具 Cargo 内置了包管理器，并且可以与 crates.io、私有注册中心、本地包和供应商源一起工作，这种差异得到了放大。

[Cargo Book](https://doc.rust-lang.org/cargo/) 详细介绍了 Cargo。

## C 和 C++ 系统库的软件包

许多 C 库在 crates.io 上提供了低级绑定和高级安全的 Rust 抽象。例如，对于 libgit2 库，既有低级的 [libgit2-sys crate](https://crates.io/crates/libgit2-sys)，也有高级的 [git2 crate](https://crates.io/crates/git2)。有关如何定义这些 crate 的更多信息，请参阅 Rust FFI 章节内容。

## 构建 C、C++ 和 Rust 代码

Cargo 的 [构建脚本](https://doc.rust-lang.org/cargo/reference/build-script-examples.html) 可以用于在 Rust 项目中构建 C 和 C++ 代码。Cargo 书籍的链接章节包括处理 C、C++ 和其他代码编译的资源链接，以及与 `pkg-config` 等一起工作的信息。

## 测试（CTest）

Cargo 支持运行 [测试](https://doc.rust-lang.org/cargo/guide/tests.html)。

## 打包分发（CPack）

与 CMake 提供的 CPack 不同，Cargo 并不自带用于打包分发给最终用户的工具。然而，存在第三方 Cargo 辅助工具用于打包，例如 [cargo-deb](https://crates.io/crates/cargo-deb) 用于创建 Debian 软件包，[cargo-generate-rpm](https://crates.io/crates/cargo-generate-rpm) 用于创建 RPM 软件包，以及 [cargo-wix](https://crates.io/crates/cargo-wix) 用于创建 Windows 安装程序。

[点击此处给我们关于此页面的反馈](https://docs.google.com/forms/d/e/1FAIpQLScoygeNlygODY2owQ-HvU8VGx3hi50aic7ZlKCyhJ0VktjiCg/viewform?usp=pp_url&entry.1450251950=构建系统（例如，CMake）)
