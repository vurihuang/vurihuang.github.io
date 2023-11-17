+++
title = "Emacs 之路"
author = ["vuri"]
lastmod = 2023-11-17T10:49:12+08:00
tags = ["emacs"]
categories = ["emacs"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [[Deprecated] Setup with Spacemacs](#deprecated-setup-with-spacemacs)
- [Setup with Doomemacs](#setup-with-doomemacs)
    - [配置使用 chemacs2(可选但是推荐)](#配置使用-chemacs2--可选但是推荐)
    - [安装 doomemacs](#安装-doomemacs)
- [Org mode](#org-mode)
    - [如何创建代码段](#如何创建代码段)
    - [如何编辑代码段](#如何编辑代码段)
    - [如何创建表格](#如何创建表格)
    - [TOC(Table of Contents)](#toc--table-of-contents)
- [Markdown](#markdown)
    - [生成 TOC](#生成-toc)
    - [预览](#预览)
- [快捷方式](#快捷方式)
- [Plugins](#plugins)
    - [Treemacs](#treemacs)

</div>
<!--endtoc-->


## [Deprecated] Setup with Spacemacs {#deprecated-setup-with-spacemacs}

```shell
  # 下载 emacs
  $ brew install --cask emacs

  # 推荐这个分支的 emacs 发行版，提供更好的原生 GUI 支持，下载对应所需的 release 版本
  # 解决 GUI 下的闪屏问题，更加丝滑
  # https://github.com/railwaycat/homebrew-emacsmacport

  # 启用 spacemacs 作为 emacs 的加载入口
  $ git clone https://github.com/syl20bnr/spacemacs spacemacs.d
  $ ln -s ~/dotfiles/spacemacs.d ~/.emacs.d

  # 启动 emacs（GUI模式），下载依赖
  $ emacs
  # 终端模式下：
  # $ emacs --nw

  # 将 spacemacs 的启动配置存到到自定义的 dotfiles 下，方便统一管理
  $ mkdir -p ~/dotfiles/.emacs.d
  $ mv ~/.spacemacs* ~/dotfiles/.emacs.d
  $ ln -s ~/dotfiles/.emacs.d/.spacemacs ~/.spacemacs
  $ ln -s ~/dotfiles/.emacs.d/.spacemacs.env ~/.spacemacs.env
```


## Setup with Doomemacs {#setup-with-doomemacs}

[Doomemacs](https://github.com/doomemacs/doomemacs): About An Emacs framework for the stubborn martian hacker.

> 为什么要使用 Doomemacs?

1.  它真的很快: 得益于它的结构设计和懒加载,比其他框架快很多;
2.  比较接近原生: 接近原生的好处是上手更好理解和体验,不需要你过多了解框架的内容(~~spacemacs~~);


### 配置使用 chemacs2(可选但是推荐) {#配置使用-chemacs2--可选但是推荐}

[Chemacs2](https://github.com/plexus/chemacs2): Emacs version switcher, improved

> 在安装使用之前,为什么要用 chemacs2?

`chemacs2` 可以很方便地在多个 Emacs profile 配置进行切换;
假定我们有一套不使用任何框架的原始配置(default profile); 一套 spacemacs 配置(spacemacs profile); 一套 doomemacs 配置(doomemacs profile):

```shell
# 1. 备份当前正在使用的配置,以 default profile 为例:
$ mv ~/.emacs.d ~/.emacs.default

# 2. 你可能原来使用了 spacemacs 配置,可以统一下风格,这里不是强制的
$ mv ~/.spacemacs.d ~/.emacs.spacemacs

# 3. 安装 chemacs2
$ git clone https://github.com/plexus/chemacs2.git ~/.emacs.d
```

编辑 `~/.emacs-profiles.el` 进行配置:

```emacs-lisp
(("default" . ((user-emacs-directory . "~/.emacs.default"))) ;; GUI 默认使用 default 配置
 ("spacemacs" . ((user-emacs-directory . "~/.emacs.spacemacs"))))
```

如果想支持多版本的 `spacemacs`,也可以这么配置:

```emacs-lisp
;; 默认场景
(("spacemacs" . ((user-emacs-directory . "~/spacemacs")
                (env . (("SPACEMACSDIR" . "~/.spacemacs.d")))))

;; 使用开发版本,原配置
("spacemacs-develop" . ((user-emacs-directory . "~/spacemacs.develop")
                    (env . (("SPACEMACSDIR" . "~/.spacemacs.d")))))

;; 使用开发版本,开发配置
("spacemacs-dev" . ((user-emacs-directory . "~/spacemacs.develop")
                (env . (("SPACEMACSDIR" . "~/.spacemacs.d.dev"))))))
```

如何使用:

```shell
# 使用默认配置 default
$ emacs

# 指定配置,等效于上面
$ emacs --with-profile default

# 指定另外一套配置
$ emacs --with-profile spacemacs
```


### 安装 doomemacs {#安装-doomemacs}

安装方式可以参考官方的:[doomemacs#Install](https://github.com/doomemacs/doomemacs#install),根据个人习惯我进行了一些调整:

```shell
# clone the repo.
$ git clone --depth 1 https://github.com/doomemacs/doomemacs ~/dotfiles/.doomemacs.d
# exports the bin path.
$ echo 'export PATH="$HOME/dotfiles/doomemacs.d/bin:$PATH"' >> ~/.aliases && source ~/.aliases
# install the deps.
$ doom install
$ mv ~/.doom.d ~/dotfiles/.doom.d
$ ln -s $HOME/dotfiles/.doom.d $HOME/.doom.d
```

添加如下内容到 `~/.emacs-profiles.el` 中:

```emacs-lisp
(("default"   . ((user-emacs-directory . "~/dotfiles/doomemacs.d/")))
 ("spacemacs"   . ((user-emacs-directory . "~/dotfiles/spacemacs.d/")))
 ("doom"   . ((user-emacs-directory . "~/dotfiles/doomemacs.d/")))
 ("legacy"   . ((user-emacs-directory . "~/dotfiles/.emacs.legacy/"))))
```


## Org mode {#org-mode}

---

Refs:

-   [org-syntax](https://orgmode.org/worg/org-syntax.html)


### 如何创建代码段 {#如何创建代码段}

输入 `src` + `TAB` 生成如下格式的代码段:

```org
#+begin_src

#+end_src
```

或者输入 `quote` + `TAB` 生成如下格式的引用:

```text
#+begin_quote

#+end_quote
```


### 如何编辑代码段 {#如何编辑代码段}

`C-c '` 进入编辑代码段界面， `C-c C-c` 保存修改， `C-c C-k` 撤销修改。

---

Refs:

-   [Structure of Code Blocks](https://orgmode.org/manual/Structure-of-Code-Blocks.html)
-   [setup-with-emacs-and-org-mode](https://andreyor.st/posts/2022-10-16-my-blogging-setup-with-emacs-and-org-mode/)


### 如何创建表格 {#如何创建表格}

Org 可以很方便通过 ASCII 来创建表格. 通过 `|` 符号作为列的分割符; 键入 `|` + `TAB` 作为表格的列;键入 `|-` 作为表格的行分割符号;表格大概长这个样子:

```org
| Name  | Phone | Age |
|-------+-------+-----|
| Peter |  1234 |  17 |
| Anna  |  4321 |  25 |
```

当你输入 `|Name|Phone|Age` 后,执行 `C-c RET`,可以直接快速生成这个样子的表格:

```org
| Name | Phone | Age |
|------+-------+-----|
|      |       |     |
```

---

Refs:

-   [Built-in Table Editor](https://orgmode.org/manual/Built_002din-Table-Editor.html)


### TOC(Table of Contents) {#toc--table-of-contents}

---

Refs:

-   [Table-of-Contents](https://orgmode.org/manual/Table-of-Contents.html)


## Markdown {#markdown}


### 生成 TOC {#生成-toc}

执行 `SPC SPC markdown-toc-generate-toc RET`


### 预览 {#预览}

1.  安装 `vmd`
    ```shell
           npm install -g vmd
    ```
2.  配置实时预览引擎
    ```emacs-lisp
            dotspacemacs-configuration-layers '(
              (markdown :variables markdown-live-preview-engine 'vmd))
    ```


## 快捷方式 {#快捷方式}

| Command                 | Key shortcut(native/Doom) | Description |
|-------------------------|---------------------------|-------------|
| org-insert-link         | C-c C-l / SPC m l l       | 插入超链接  |
| org-toggle-link-display | SPC m l t                 | 展示/隐藏超链接 |


## Plugins {#plugins}


### Treemacs {#treemacs}

Repo: [Treemacs](https://github.com/Alexander-Miller/treemacs)

> a tree layout file explorer for Emacs


#### 如何使用 {#如何使用}

spacemacs 已经包含 treemacs layer，可以直接使用：

```emacs-lisp
dotspacemacs-configuration-layers
'(treemacs :variables
  treemacs-use-git-mode 'deferred)
```

-   添加工程（Project）到工作空间（Workspace）中

    光标焦点移动到 Treemacs 窗口中，执行 `C-c C-p a` 或者 `SPC SPC treemacs-add-project RET`
    选择指定目录到工程中。

-   对工作空间的工程进行排序

    执行 `C-c C-w e` 或者 `SPC SPC treemacs-edit-workspaces RET` ，会弹出窗口对文件进行编辑：

<!--listend-->

```org
#+TITLE: Edit Treemacs Workspaces & Projects
# Call =treemacs-finish-edit= or press =C-c C-c= when done.
# [[https://github.com/Alexander-Miller/treemacs#conveniently-editing-your-projects-and-workspaces][Click here for detailed documentation.]]
# To cancel you can simply kill this buffer.

* Default
** dotfiles
    - path :: ~/dotfiles
```

确认编辑修改后， `C-c C-c` 进行保存并退出。
