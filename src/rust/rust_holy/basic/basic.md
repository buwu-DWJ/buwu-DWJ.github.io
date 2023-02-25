# Rust基础知识

在线运行测试
```rust
fn main(){
    println!("Hello, world!")
}
```

## 安装rust(202302)

### macOS

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```
显示
```console
Rust is installed now. Great!
```
则安装成功!

安装C语言编译器

```console
$ xcode-select --install
```

### Windows

先安装Microsoft C++ Build Tools, 勾选安装C++环境. 然后将Rust所需的msvc命令行程序手动添加到环境变量中, 其位于`%Visual Studio 安装位置%\VC\Tools\MSVC\%version%\bin\Hostx64\x64`下.

在[RUSTUP-INIT](https://www.rust-lang.org/learn/get-started)下载系统对应的版本,
```console
PS C:\Users\Hehongyuan> rustup-init.exe
......
Current installation options:

   default host triple: x86_64-pc-windows-msvc
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation

```

### windows arm

[rust other_installation](https://forge.rust-lang.org/infra/other-installation-methods.html)  `aarch64-pc-windows-msvc`
安装目录默认为`C:\Program Files(x86)\Rust stable MSVC 1.67\`



## cargo

上面的命令使用 `cargo new` 创建一个项目，项目名是 `world_hello`, 该项目的结构和配置文件都是由 `cargo` 生成，意味着**我们的项目被 `cargo` 所管理**.

下面来看看创建的项目结构：

```console
$ tree
.
├── .git
├── .gitignore
├── Cargo.toml
└── src
    └── main.rs

```

## 运行项目

有两种方式可以运行项目：

1. `cargo run`

2. 手动编译和运行项目

首先来看看第一种方式，在之前创建的 `world_hello` 目录下运行：

```console
$ cargo run
   Compiling world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/world_hello`
Hello, world!
```

好了，你已经看到程序的输出：`"Hello, world"`。

如果你安装的 Rust 的 `host triple` 是 `x86_64-pc-windows-msvc` 并确认 Rust 已经正确安装，但在终端上运行上述命令时，出现类似如下的错误摘要 `` linking with `link.exe` failed: exit code: 1181 ``，请使用 Visual Studio Installer 安装 `Windows SDK`。

上述代码，`cargo run` 首先对项目进行编译，然后再运行，因此它实际上等同于运行了两个指令，下面我们手动试一下编译和运行项目：

编译

```console
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
```

运行

```console
$ ./target/debug/world_hello
Hello, world!
```

行云流水，但谈不上一气呵成。在调用的时候，路径 `./target/debug/world_hello` 中有一个明晃晃的 `debug` 字段，没错我们运行的是 `debug` 模式，在这种模式下，**代码的编译速度会非常快**，可是福兮祸所依，**运行速度就慢了**. 原因是，在 `debug` 模式下，Rust 编译器不会做任何的优化，只为了尽快的编译完成，让开发流程更加顺畅。

想要高性能的代码怎么办？ 简单，添加 `--release` 来编译：

- `cargo run --release`
- `cargo build --release`

试着运行一下我们高性能的 `release` 程序：

```console
$ ./target/release/world_hello
Hello, world!
```

## cargo check

当项目大了后，`cargo run` 和 `cargo build` 不可避免的会变慢，那么有没有更快的方式来验证代码的正确性呢？

`cargo check` 是我们在代码开发过程中最常用的命令，它的作用很简单：快速的检查一下代码能否编译通过。因此该命令速度会非常快，能节省大量的编译时间。

```console
$ cargo check
    Checking world_hello v0.1.0 (/Users/sunfei/development/rust/world_hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
```

> Rust 虽然编译速度还行，但是还是不能与 Go 语言相提并论，因为 Rust 需要做很多复杂的编译优化和语言特性解析，甚至连如何优化编译速度都成了一门学问: [优化编译速度](https://course.rs/profiling/compiler/speed-up.html)


## Cargo.toml 和 Cargo.lock

`Cargo.toml` 和 `Cargo.lock` 是 `cargo` 的核心文件，它的所有活动均基于此二者。

- `Cargo.toml` 是 `cargo` 特有的**项目数据描述文件**。它存储了项目的所有元配置信息，如果 Rust 开发者希望 Rust 项目能够按照期望的方式进行构建、测试和运行，那么，必须按照合理的方式构建 `Cargo.toml`。

- `Cargo.lock` 文件是 `cargo` 工具根据同一项目的 `toml` 文件生成的**项目依赖详细清单**，因此我们一般不用修改

> 什么情况下该把 `Cargo.lock` 上传到 git 仓库里？很简单，当你的项目是一个可运行的程序时，就上传 `Cargo.lock`，如果是一个依赖库项目，那么请把它添加到 `.gitignore` 中

现在用 VSCode 打开上面创建的"世界，你好"项目，然后进入根目录的 `Cargo.toml` 文件，可以看到该文件包含不少信息：

### package 配置段落

`package` 中记录了项目的描述信息，典型的如下：

```toml
[package]
name = "world_hello"
version = "0.1.0"
edition = "2021"
```

`name` 字段定义了项目名称，`version` 字段定义当前版本，新项目默认是 `0.1.0`，`edition` 字段定义了使用的 Rust 大版本

### 定义项目依赖

使用 `cargo` 工具的最大优势就在于，能够对该项目的各种依赖项进行方便、统一和灵活的管理。

在 `Cargo.toml` 中，主要通过各种依赖段落来描述该项目的各种依赖项：

- 基于 Rust 官方仓库 `crates.io`，通过版本说明来描述
- 基于项目源代码的 git 仓库地址，通过 URL 来描述
- 基于本地项目的绝对路径或者相对路径，通过类 Unix 模式的路径来描述

这三种形式具体写法如下：

```toml
[dependencies]
rand = "0.3"
hammer = { version = "0.5.0"}
color = { git = "https://github.com/bjz/color-rs" }
geometry = { path = "crates/geometry" }
```