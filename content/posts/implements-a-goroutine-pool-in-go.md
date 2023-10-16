+++
title = "Golang 实现一个协程池"
author = ["vuri"]
lastmod = 2023-10-16T12:15:49+08:00
tags = ["golang"]
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [为什么要使用 goroutine 协程池](#为什么要使用-goroutine-协程池)
- [如何实现一个简易 goroutine 协程池](#如何实现一个简易-goroutine-协程池)

</div>
<!--endtoc-->


## 为什么要使用 goroutine 协程池 {#为什么要使用-goroutine-协程池}

1.  在并发编程时，可以限制 goroutine 的数量，复用资源，提升性能；
2.  保持 CPU 缓存命中率，让 CPU 缓存处于活跃状态；


## 如何实现一个简易 goroutine 协程池 {#如何实现一个简易-goroutine-协程池}

1.  先对我们的目标进行抽象，池化的对象无非是启动、停止、提交任务：

<!--listend-->

```go
type WorkerPool struct {
}

func (wp *WorkerPool) Start() {

}

func (wp *WorkerPool) Stop() {

}

func (wp *WorkerPool) Submit(fn func()) error {
  panic("implement me")
}
```

---

Refs:

-   [fasthttp workerpool](https://github.com/valyala/fasthttp/blob/master/workerpool.go)
-   [rulego workerpool](https://github.com/rulego/rulego/blob/main/pool/workerpool.go)
-   [ants: a high-performance and low-cost goroutine pool](https://github.com/panjf2000/ants)
