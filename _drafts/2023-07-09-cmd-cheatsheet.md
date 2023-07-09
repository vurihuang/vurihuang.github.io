---
layout: post
title: Command cheatsheet
slug: command-cheatsheet
---

<!-- vim-markdown-toc GFM -->

* [tar](#tar)
* [tcpdump](#tcpdump)

<!-- vim-markdown-toc -->


## tar

``` sh
# 解压压缩包文件到指定文件夹
$ mkdir dirname
$ tar -zxvf archive.tar.gz -C dirname --strip-components=1
```

## tcpdump

``` sh
## 列出所有网卡接口
$ tcpdump -D
````
