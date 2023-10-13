+++
title = "Shell 速查表"
author = ["vuri"]
lastmod = 2023-10-14T01:16:27+08:00
categories = ["cheatsheet"]
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [tar](#tar)
- [tcpdump](#tcpdump)

</div>
<!--endtoc-->


## tar {#tar}

```shell
# 解压压缩包文件到指定文件夹
mkdir dirname
tar -zxvf archive.tar.gz -C dirname --strip-components=1
```


## tcpdump {#tcpdump}

```shell
# 列出所有网卡接口
tcpdump -D
```
