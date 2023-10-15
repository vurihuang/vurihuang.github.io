+++
title = "Shell 速查表"
author = ["vuri"]
lastmod = 2023-10-15T01:11:17+08:00
tags = ["cheatsheet"]
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
mkdir ${dirname}
tar -zxvf archive.tar.gz -C ${dirname} --strip-components=1
```


## tcpdump {#tcpdump}

列出所有网卡接口

```shell
tcpdump -D
```

抓取指定网卡 `eht1` 按照每个文件大小 10M输出到文件 `dump.cap`

```shell
$ tcpdump -nni eth1 -w dump.cap -C10M -Zroot
```
