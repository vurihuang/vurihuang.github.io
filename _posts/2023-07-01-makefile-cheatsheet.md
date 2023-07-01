---
layout: post
title: Makefile 速查表
slug: makefile-cheatsheet
---


## Missing separator

`make` 使用 `tab` 来进行缩进，用空格缩进则报错；如果是 `tab`，会以 `^I` 符号呈现，可以通过以下方式进行检查：

``` sh
$ cat Makefile
.PHONY: print
print:
  echo "test"
  echo "test"

$ make dryrun
Makefile:3: *** missing separator.  Stop.

$ cat -e -t -v Makefile
.PHONY: print$
print:$
  echo "test"$
^Iecho "test"$
```

---
Refs:
- [make规则介绍](https://www.gnu.org/software/make/manual/make.html#Rule-Introduction)
