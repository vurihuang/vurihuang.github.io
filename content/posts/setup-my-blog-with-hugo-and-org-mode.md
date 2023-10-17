+++
title = "Setup My Blog with Hugo and Org Mode"
author = ["vuri"]
lastmod = 2023-10-17T00:29:50+08:00
tags = ["emacs", "orgmode"]
categories = ["emacs"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [安装 Hugo](#安装-hugo)
- [项目初始化](#项目初始化)
- [修改配置文件 hugo.toml](#修改配置文件-hugo-dot-toml)
- [创建第一篇 Hello World 文章](#创建第一篇-hello-world-文章)
- [使用 org-mode 来编辑博客](#使用-org-mode-来编辑博客)

</div>
<!--endtoc-->


## 安装 Hugo {#安装-hugo}

```shell
$ brew install hugo
```


## 项目初始化 {#项目初始化}

```shell
$ hugo new site blog
$ cd blog; git init .
# 安装主题
$ git submodule add https://github.com/luizdepra/hugo-coder.git themes/hugo-coder
```


## 修改配置文件 hugo.toml {#修改配置文件-hugo-dot-toml}

```toml
baseurl = "http://www.example.com"
title = "example"
theme = "hugo-coder"
languagecode = "en"
defaultcontentlanguage = "en"

paginate = 20

[markup.highlight]
style = "github-dark"

[params]
  author = "example"
  info = ""
  description = ""
  keywords = "blog,developer,personal"
  avatarurl = "images/avatar.jpg"
  #gravatar = "john.doe@example.com"

  faviconSVG = "/img/favicon.svg"
  favicon_32 = "/img/favicon-32x32.png"
  favicon_16 = "/img/favicon-16x16.png"

  since = 2020

  enableTwemoji = true

  colorScheme = "auto"
  hidecolorschemetoggle = false

  # customCSS = ["css/custom.css"]
  # customSCSS = ["scss/custom.scss"]
  # customJS = ["js/custom.js"]

[taxonomies]
  category = "categories"
  series = "series"
  tag = "tags"
  author = "authors"

# Social links
[[params.social]]
  name = "Github"
  icon = "fa fa-github fa-2x"
  weight = 1
  url = "https://github.com/example/"

# Menu links
[[menu.main]]
  name = "Blog"
  weight = 1
  url  = "posts/"
[[menu.main]]
  name = "About"
  weight = 2
  url = "about/"
```


## 创建第一篇 Hello World 文章 {#创建第一篇-hello-world-文章}

```shell
$ hugo new content posts/hello-world.md
$ cat content/posts/hello-world.md
```

显示如下内容：

```markdown
+++
title = 'Hello World'
date = 2023-10-14T01:31:21+08:00
draft = true
+++
```

在文本中追加内容 `hello world` ，启动 Hugo Server：

```shell
$ echo 'hello world' >> content/posts/hello-world.md
# 同时构建草稿文章
$ hugo server --buildDrafts
# ...
# Web Server is available at http://localhost:62743/ (bind address 127.0.0.1)
# ...
```

打开浏览器，访问 `http://localhost:62743/` ：

{{< figure src="/images/hello-world.png" >}}


## 使用 org-mode 来编辑博客 {#使用-org-mode-来编辑博客}

1.  使用 `ox-hugo` 插件来支持 org 文件生成 markdown 文件：
    spacemacs 已经集成 `ox-hugo` 插件，直接启用即可：
    ```emacs-lisp
    dotspacemacs-configuration-layers
    '(org :variables
          org-enable-hugo-support t)
    )
    ```

2.  在博客根目录下创建 org 文件，例如： `index.org`
    ```org
    #+title: Example's blog
    #+author: nobody

    #+hugo_auto_set_lastmod: t
    #+hugo_base_dir: .
    #+hugo_section: .

    #+options: toc:2

    * Posts
    :properties:
    :export_hugo_section: posts
    :end:

    ** Hello world!
    :properties:
    :export_file_name: hello-world
    :end:

    Hello, this is my first article.
    ```
    执行 `, e e` 或 `SPC SPC org-export-dispatch RET` 会看到如下窗口，再执行 `H H` 导出为 markdown 文件，并保存到 `content/posts` 目录下：

    {{< figure src="/images/org-export-dispatch-window.png" >}}

3.  保存后自动导出生成 markdown 文件

    每次执行 `, e e H H` 生成操作还挺繁琐，如何进行配置每次一保存 org 文件自动生成导出呢？

    在博客根目录下创建 `.dir-locals.el` 文件：
    ```emacs-lisp
    ((org-mode . ((eval . (org-hugo-auto-export-mode)))))
    ```
