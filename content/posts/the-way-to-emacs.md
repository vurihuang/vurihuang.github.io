+++
title = "Emacs 之路"
author = ["vuri"]
lastmod = 2023-11-09T01:38:54+08:00
tags = ["emacs"]
categories = ["emacs"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Setup with spacemacs](#setup-with-spacemacs)
- [Org mode](#org-mode)
    - [语法](#语法)
    - [如何创建代码段](#如何创建代码段)
    - [如何编辑代码段](#如何编辑代码段)
    - [TOC(Table of Contents)](#toc--table-of-contents)
- [Markdown](#markdown)
    - [生成 TOC](#生成-toc)
    - [预览](#预览)
- [Plugins](#plugins)
    - [Treemacs](#treemacs)

</div>
<!--endtoc-->


## Setup with spacemacs {#setup-with-spacemacs}

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


## Org mode {#org-mode}


### 语法 {#语法}

---

Refs:

-   [org-syntax](https://orgmode.org/worg/org-syntax.html)


### 如何创建代码段 {#如何创建代码段}

执行 `, i b` 弹出菜单选择框，再按 s `s[src]`

```org
#+BEGIN_SRC org
#+END_SRC
```


### 如何编辑代码段 {#如何编辑代码段}

`C-c '` 进入编辑代码段界面， `, c` 保存修改， `, k` 撤销修改。

---

Refs:

-   [Structure of Code Blocks](https://orgmode.org/manual/Structure-of-Code-Blocks.html)
-   [setup-with-emacs-and-org-mode](https://andreyor.st/posts/2022-10-16-my-blogging-setup-with-emacs-and-org-mode/)


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
    ```org
    #+TITLE: Edit Treemacs Workspaces & Projects
    # Call ~treemacs-finish-edit~ or press ~C-c C-c~ when done.
    # [[https://github.com/Alexander-Miller/treemacs#conveniently-editing-your-projects-and-workspaces][Click here for detailed documentation.]]
    # To cancel you can simply kill this buffer.

    * Default
    ** dotfiles
    ​ - path :: ~/dotfiles
    ```
    确认编辑修改后， `C-c C-c` 进行保存并退出。
