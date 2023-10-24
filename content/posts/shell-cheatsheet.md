+++
title = "Shell 速查表"
author = ["vuri"]
lastmod = 2023-10-25T01:39:15+08:00
tags = ["cheatsheet"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [dig](#dig)
- [dnsperf](#dnsperf)
- [journalctl](#journalctl)
- [netstat](#netstat)
- [nslookup](#nslookup)
- [tar](#tar)
- [tcpdump](#tcpdump)

</div>
<!--endtoc-->


## dig {#dig}


## dnsperf {#dnsperf}

Usage: DNS 性能压测工具

Install:

```shell
# ubuntu
$ apt-get install -y dnsperf

# centos
$ yum install -y dnsperf
```

Examples:

```shell
# 构造 DNS 查询记录
$ cat <<EOF >records.txt
baidu.com A
www.baidu.com A
mail.baidu.com A
github.com A
www.yahoo.com A
www.microsoft.com A
www.aliyun.com A
developer.aliyun.com A
www.tencentcloud.com A
kubernetes.io A
kubernetes A
kubernetes.default.svc.cluster.local A
kube-dns.kube-system.svc.cluster.local A
EOF


$ dnsperf -l 10 -s 127.0.0.1 -Q 100 -d records.txt
```


## journalctl {#journalctl}

Usage: 查看 CentOS 系统 systemd 日志

```shell
# 获取系统启动记录

$ journalctl --list-boots
-1 6436baaf9ffd4122bf6ff4704e87de27 Sun 2023-10-22 15:49:49 CST—Mon 2023-10-23 10:41:47 CST
0 b1b0fbdda6344d91815684bd6a910281 Mon 2023-10-23 10:44:09 CST—Mon 2023-10-23 14:35:29 CST

# 显示本次启动的信息
$ journalctl -b -0

# 显示上次启动的信息
$ journalctl -b -1

# 显示自xx时间点开始的日志
$ journalctl --since="2023-10-21 17:00:00"

# 显示自x时间点到y时间点的日志

$ journalctl --since="2023-10-21 17:23:29" --util="2023-10-21 18:00:00"
```


## netstat {#netstat}

Usage:

Install:

```shell
# ubuntu
$ apt-get install -y net-tools
```


## nslookup {#nslookup}

Usage: 域名解析查询

Install:

```shell
# centos
$ yum install -y bind-utils
```

Examples:

```shell
$ nslookup www.baidu.com

# 指定查询 A 记录
$ nslookup -type=a www.baidu.com
```


## tar {#tar}

```shell
# 解压压缩包文件到指定文件夹
mkdir ${dirname}
tar -zxvf archive.tar.gz -C ${dirname} --strip-components=1
```


## tcpdump {#tcpdump}

Install:

```shell
# install on ubuntu
$ apt-get install -y dnsutils

# install on centos
$ yum install -y bind-utils
```

Examples:

```shell
# 列出所有网卡接口：
tcpdump -D

# 抓取指定网卡 =eht1= ，按照每个文件大小10M输出抓包内容到文件 =dump.cap= 中：
$ tcpdump -nni eth1 -w dump.cap -C10M -Zroot

$ tcpdump -nnn -i any src host 127.0.0.1 and port 53 -tttt -Zroot
```
