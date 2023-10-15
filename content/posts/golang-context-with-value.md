+++
title = "Golang Context with value"
author = ["vuri"]
date = 2020-08-20
lastmod = 2023-10-14T02:14:27+08:00
categories = ["golang"]
draft = false
+++

今天遇到个很有意思的一段代码，这段程序会打印出什么结果：

```go
package main

import (
  "context"
  "fmt"
)

func f(ctx context.Context) {
  context.WithValue(ctx, "foo", -6)
}

func main() {
  ctx := context.TODO()
  f(ctx)
  fmt.Println(ctx.Value("foo"))
  // -6
  // 0
  // <nil>
  // panic
}
```

先让我们看看 `context.TODO()` 返回的结果是什么：

```go
var (
  background = new(emptyCtx)
  todo       = new(emptyCtx)
)

type emptyCtx int

func TODO() Context {
  return todo
}
```

`context.TODO()` 返回的实例返回的正是一个 `emptyCtx` 对象，也就是 `int` ，它不能被 cancel，也不包含任何值，并且也没有 deadline。同时也不是一个空的结构体 `struct{}` ，因为它需要一个目标地址。

那么 `context.WithValue` 做了些什么事情呢：

```go
type WithValue(parent Context, key, val interface{}) Context {
    if parent == nil {
        panic("cannot create context from nil parent")
    }
    if key == nil {
        panic("nil key")
    }
    if !reflectlite.TypeOf(key).Comparable() {
        panic("key is not comparable")
    }
    return &valueCtx{parent, key, val}
}

func valueCtx struct {
    Context,
    key, val interface{}
}
```

看到这里其实我们一开始的程序的结果已经很明显了，~WithValue~ 每次都会返回一个新的带有 key-value 值的上下文对象 `valueCtx` ，如果没有重新赋值，那么我们的 key-value 就会被丢失，并不会携带下去。

那么 `context.Value` 是怎么查找值的呢：

```go
func (c *valueCtx) Value(key interface{}) interface{} {
  if c.key == key {
    return c.val
  }
  return c.Context.Value(key)
}
```

在查找指定 key 时，会先从当前的 context 对象中查看是否存在对应的 key，没有的话则回溯到 parent context 进行查找，那么什么时候是查找的尽头呢：

```go
func (*emptyCtx) Value(key interface{}) interface{} {
  return nil
}
```

查找的尽头正是当 context 是一开始的 `emptyCtx` 空实现上下文对象时。

也正是因为 `valueCtx` 的实现如上面这样，是一种嵌套的结构，并且每次都是生成一个新的对象，官方的建议在使用时应该只传递必要的参数，来减少它的层级和数据的大小：

```text
WithValue returns a copy of parent in which the value associated with key is val.
Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.
```
