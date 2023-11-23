+++
title = "The hardway to Rust"
author = ["vuri"]
lastmod = 2023-11-17T14:22:37+08:00
tags = ["Rust"]
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [00-Setup](#00-setup)
    - [Install](#install)
    - [Emacs/Spacemacs](#emacs-spacemacs)
    - [配置 rust-analyzer](#配置-rust-analyzer)
    - [配置使用镜像源](#配置使用镜像源)
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


### 配置使用镜像源 {#配置使用镜像源}

创建 `~/.cargo/config` 文件，以 `rsproxy` 作为示例：

```toml
  [source.crates-io]
  replace-with = 'rsproxy'

  [source.rsproxy]
  registry = "https://rsproxy.cn/crates.io-index"

  [source.rsproxy-sparse]
  registry = "sparse+https://rsproxy.cn/index/"

  [registries.rsproxy]
  index = "https://rsproxy.cn/crates.io-index"

  [net]
  git-fetch-with-cli = true
```

使用 [crm (Cargo registry manager)](https://github.com/wtklbm/crm) 进行镜像源管理：

```shell
  # 安装
  $ cargo install crm

  $ crm
  命令无效。参考:

  crm best                    评估网络延迟并自动切换到最优的镜像
  crm best git                仅评估 git 镜像源
  crm best sparse             仅评估支持 sparse 协议的镜像源
  crm best git-download       仅评估能够快速下载软件包的 git 镜像源 (推荐使用)
  crm best sparse-download    仅评估能够快速下载软件包且支持 sparse 协议的镜像源 (推荐使用)
  crm current                 获取当前所使用的镜像
  crm default                 恢复为官方默认镜像
  crm install [args]          使用官方镜像执行 "cargo install"
  crm list                    从镜像配置文件中获取镜像列表
  crm publish [args]          使用官方镜像执行 "cargo publish"
  crm remove <name>           在镜像配置文件中删除镜像
  crm save <name> <addr> <dl> 在镜像配置文件中添加/更新镜像
  crm test [name]             下载测试包以评估网络延迟
  crm update [args]           使用官方镜像执行 "cargo update"
  crm use <name>              切换为要使用的镜像
  crm version                 查看当前版本
  crm check-update            检测版本更新

  # 获取镜像列表
  $ crm list
  bfsu           - https://mirrors.bfsu.edu.cn/git/crates.io-index.git
  bfsu-sparse    - sparse+https://mirrors.bfsu.edu.cn/crates.io-index/
  hit            - https://mirrors.hit.edu.cn/crates.io-index.git
  nju            - https://mirror.nju.edu.cn/git/crates.io-index.git
  rsproxy        - https://rsproxy.cn/crates.io-index
  rsproxy-sparse - sparse+https://rsproxy.cn/index/
  * rust-lang    - https://github.com/rust-lang/crates.io-index
  sjtu           - https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index
  sjtu-sparse    - sparse+https://mirrors.sjtug.sjtu.edu.cn/crates.io-index/
  tuna           - https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git
  tuna-sparse    - sparse+https://mirrors.tuna.tsinghua.edu.cn/crates.io-index/
  ustc           - git://mirrors.ustc.edu.cn/crates.io-index
  ustc-sparse    - sparse+https://mirrors.ustc.edu.cn/crates.io-index/

  # 自动选择最优镜像源
  $ crm best
  已切换到 sjtu 镜像源

  # 查看当前使用的镜像源
  $ crm current
  sjtu: https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index
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
