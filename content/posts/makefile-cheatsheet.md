+++
title = "Makefile 速查表"
author = ["vuri"]
lastmod = 2023-11-23T03:43:38+08:00
tags = ["cheatsheet"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Missing separator](#missing-separator)
- [在 Makefile 中使用 $ 符号](#在-makefile-中使用-符号)

</div>
<!--endtoc-->


## Missing separator {#missing-separator}

`make` 使用 `tab` 来进行缩进，用空格缩进则报错；如果是 `tab` ，会以 `^I` 符号呈现，可以通过以下方式进行检查：

```shell
  $ cat Makefile
  .PHONY: print
  print:
  echo "test"
  echo "test"

  $ make print
  Makefile:3: *** missing separator.  Stop.

  $ cat -e -t -v Makefile
  .PHONY: print$
  print:$
  echo "test"$
  ^Iecho "test"$
```


## 在 Makefile 中使用 $ 符号 {#在-makefile-中使用-符号}

在 Makefile 中，需要区分 `$` 的使用，是为了引用变量例如 `$variable` ；还是需要使用 `$` 传递给 shell 命令；因此如果单纯想要使用 `$` ，就需要重复写两遍 `$$` 进行转义：

```makefile
  # 无效用法：
  .PHONY: replace
    echo ' foo' | sed 's/foo$/bar/g'

  # 正确用法：
  .PHONY: replace
    echo ' foo' | sed 's/foo$$/bar/g'
```

---

Refs:

-   [make规则介绍](https://www.gnu.org/software/make/manual/make.html#Rule-Introduction)
-   [Using Variables](https://www.gnu.org/software/make/manual/make.html#Using-Variables)
-   [make macros](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/make.html)
