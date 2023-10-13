+++
title = "Setup My Blog with Hugo and Org Mode"
author = ["vuri"]
lastmod = 2023-10-14T01:59:09+08:00
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [安装 Hugo](#安装-hugo)
- [项目初始化](#项目初始化)
- [修改配置文件 hugo.toml](#修改配置文件-hugo-dot-toml)
- [创建第一篇 Hello World 文章](#创建第一篇-hello-world-文章)

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

在文本中追加内容 `Hello World!` ，并启动 Hugo Server：

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
