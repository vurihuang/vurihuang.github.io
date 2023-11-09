+++
title = "The hardway to Rust"
author = ["vuri"]
lastmod = 2023-11-09T16:03:13+08:00
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [00-Setup](#00-setup)
    - [Install](#install)
    - [Emacs/Spacemacs](#emacs-spacemacs)
    - [配置 rust-analyzer](#配置-rust-analyzer)
- [01-Hello world](#01-hello-world)
- [02-Hello Cargo](#02-hello-cargo)

</div>
<!--endtoc-->


## 00-Setup {#00-setup}


### Install {#install}

`rustup` 是 Rust 的安装程序，通过下面这个方式进行安装:

```shell
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh

$ rustup -V
rustup 1.26.0 (5af9b9484 2023-04-05)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.73.0 (cc66ad468 2023-10-03)`

$ rustc -V
rustc 1.73.0 (cc66ad468 2023-10-03)

$ cargo -V
cargo 1.73.0 (9c4383fb5 2023-08-26)
```

如何进行版本更新:

```shell
$ rustup update
```

执行 `rustup doc` 打开本地文档


### Emacs/Spacemacs {#emacs-spacemacs}

在 Emacs/Spacemacs 上进行环境配置，启用 `Rust layer` ：

```emacs-lisp
(dotspacemacs-configuration-layers
 '(
   (rust :variables
         rustic-format-on-save t))
)
```


### 配置 rust-analyzer {#配置-rust-analyzer}

`rust-analyzer` 是 `Rust` 的 `LSP(Language Server Protocol)` 的实现，提供自动补全、跳转等功能。

```shell
$ brew install rust-analyzer
```


## 01-Hello world {#01-hello-world}

创建工程目录 `l01-hello-world`

```shell
$ mkdir l01-hello-world && cd l01-hello-world && touch main.rs
```

添加 `main.rs` 文件并保存如下内容：

```rust
fn main() {
    println!("Hello world!");
}
```

```shell
$ rustc main.rs
$ ./main
Hello world!
```


## 02-Hello Cargo {#02-hello-cargo}

```shell
# 创建工程
$ cargo new l02-hello-cargo

$ tree -L 2 l02-hello-cargo
l02-hello-cargo
├── Cargo.lock
├── Cargo.toml
├── README.md
├── src
│   └── main.rs
└── target

# 执行程序
$ cd l02-hello-cargo
$ cargo run
Compiling l02-hello-cargo v0.1.0
Finished dev [unoptimized + debuginfo] target(s) in 0.56s
Running `target/debug/l02-hello-cargo`
Hello, world!

# 等价于
$ cargo build
$ ./target/debug/l02-hello-cargo
Hello, world!

# 在 release 模式下，采用编译优化
$ cargo run --release
$ cargo build --release
```
