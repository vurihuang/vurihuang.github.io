+++
title = "Emacs 之路"
author = ["vuri"]
lastmod = 2023-10-14T01:23:09+08:00
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Setup with spacemacs](#setup-with-spacemacs)
- [Org mode](#org-mode)
    - [语法](#语法)
    - [如何创建代码段](#如何创建代码段)
    - [如何编辑代码段](#如何编辑代码段)
    - [TOC](#toc)
- [Markdown](#markdown)
    - [生成 TOC](#生成-toc)
    - [预览](#预览)

</div>
<!--endtoc-->


## Setup with spacemacs {#setup-with-spacemacs}

```shell
brew install emacs
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


### TOC {#toc}

---

Refs:
[Table-of-Contents](https://orgmode.org/manual/Table-of-Contents.html)


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
