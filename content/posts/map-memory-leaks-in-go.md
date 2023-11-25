+++
title = """
  探究 Go map "内存泄露"
  """
author = ["vuri"]
lastmod = 2023-11-25T11:16:59+08:00
tags = ["golang", "Trans"]
categories = ["golang"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Intro](#intro)
- [【译】Go map 内存泄露](#译-go-map-内存泄露)
- [是不是内存泄露？](#是不是内存泄露)

</div>
<!--endtoc-->


## Intro {#intro}

起因是看到这篇文章[ Maps-and-memory-leaks-in-go](https://teivah.medium.com/maps-and-memory-leaks-in-go-a85ebe6e7e69) 讲述关于 Go Map 的内存泄露问题，在 [reddit](https://www.reddit.com/r/golang/comments/xq6lm8/maps_and_memory_leaks_in_go/) 底下也有很多有意思的探讨，我们先简单翻译下原文介绍下背景，再结合 reddit 上的评论一起来分析分析。


## 【译】Go map 内存泄露 {#译-go-map-内存泄露}

{{< figure src="https://raw.githubusercontent.com/vurihuang/images/master/imgs202311211109151.png" >}}

当我们在使用 Go map 的时候，我们需要对 map 扩缩容一些重要的特性有一定的了解。咱们来看一个会造成"内存泄露"的例子：

```go
m := make(map[int][128]byte)
```

`map m` 的每个元素值是一个128字节的数组，我们会进行以下操作：

1.  初始化一个空的 map;
2.  添加一百万个元素;
3.  删除 map 的所有元素，并执行垃圾回收;

每一步执行完成，都会打印一下堆的内存分配大小，下面是一个示例：

```go
func main() {
	n := 1_000_000
	m := make(map[int][128]byte)
	printAlloc()

	for i := 0; i < n; i++ { // 添加一百万个元素
		m[i] = [128]byte{}
	}
	printAlloc()

	for i := 0; i < n; i++ { // 删除一百万个元素
		delete(m, i)
	}
	runtime.GC()
	printAlloc()
	runtime.KeepAlive(m)
}

func printAlloc() {
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
	fmt.Printf("%d KB\n", m.Alloc/1024)
}
```

我们初始了一个 map，添加了一百万个元素，删除了一百万个元素，然后执行了一次垃圾回收。同时执行了 `runtime.KeepAlive` 来保留 map m 的引用确保不会被回收，以下是执行结果：

```text
0 MB   # 当 map 被初始化后
461 MB # 当添加一百万个元素后
293 MB # 当删除一百万个元素后
```

我们观察到了什么?刚开始的时候，堆的大小还是很小的，当添加了一百万个元素的时候一下子增长了非常多。当我们期待删除这所有元素的时候会释放掉堆内存，实际情况不符合我们的预期。即使最后我们执行了一次 GC 垃圾回收来释放这些对象，堆的大小依赖还有293 MB 的空间。内存占用虽然减少了，但是表现上和我们想的不太一样。它的原因是为什么?我们需要先深入了解一下 Go 里面的 map 的机制是什么。

map 字典提供了一个无序的 key-value 键值对集合，所有的 keys 都是不重复的。在 Go 里面，map 一般基于哈希表(hash table)来实现：一个一维数组，数组里的每个元素指向存储桶的指针引用，存储桶的结构可以存放 key-value 的键值对，如下图所示：

{{< figure src="https://raw.githubusercontent.com/vurihuang/images/master/imgs202311212256313.png" >}}

图1： 一个展示第一个存储桶的哈希表示例。

每一个存储桶都是固定8个长度大小的数组。如果把一个元素插入到已经满了的存储桶则会溢出，Go 会创建另外一个存储桶也是固定8个长度大小的数组，然后把元素放进去再链接到前一个存储桶。如下图所示：

{{< figure src="https://raw.githubusercontent.com/vurihuang/images/master/imgs202311212257202.png" >}}

图2： 在存储桶溢出的例子中，Go 分配了一个新的存储桶并链接到上一个存储桶.

在 Go 中，map 实际上指向的是 `runtime.hmap` 结构体的指针。这个结构体包含了众多的字段，其中有个 `B` 字段它表示 map 中桶的个数：

[golang hmap](https://github.com/golang/go/blob/3e67f46d4f7d661504d281bdedbd1432c09bd751/src/runtime/map.go#L117)

```go
type hmap struct {
	// ...
	B uint8 // buckets 桶数量，指数值(log_2)，可以承载负载因子*2^B次方的元素集合
	// ...
}
```

当添加一百万个元素时，因为 `2^18 = 262,144` 个存储桶(262,144\*8 &gt; 100万)，所以 `B` 的值等于18。当我们删除一百万个元素的时候， `B` 的值是多少？答案还是18。因此 map 还是包含了那么多的存储桶。

归根结底 map 中的存储桶数量不会减少， 因此删除元素时不会影响 map 的存储桶数量， 它只会把存储桶中的槽位置空， map 的存储桶数量只会增长，不会减少。

在前一个例子中，我们的内存占用因为垃圾回收之后从461 MB 减少到了293 MB， 但是 map 自身的空间是不受影响的， 意味着 map 溢出桶占用的空间还会保留着。

咱们现在一起讨论下，如果 map 不会缩容的话再什么情况下会造成问题。 想象一下如果使用 `map[int][128]byte` 作为缓存， 如果这个 map 以客户的 ID(int) 为 key 保存128个字节的序列。 假设我们想保留最后1000个客户， 虽然 map 不会缩容但它还是可控的常量大小， 所以我们还不太需要担心会造成问题。假设我们需要存放一个小时的数据， 并且我们需要在黑色星期五进行节日大促销， 可能一个小时内会有一百万的客户会请求访问我们的系统。 等过了几天之后， 我们的 map 缓存这个时候仍保留了那么多的存储桶， 这也就解释了为什么我们的系统内存不断被消耗升高不会被释放。

我们怎么才能在不重启服务的前提下清理释放内存呢? 一种解决方案是定期拷贝数据并重建当前的 map。 比如说每小时拷贝一次当前的 map， 把所有元素都复制到新的 map 中，然后释放原来的 map。 这个方案主要的缺点就是从复制到被垃圾回收之前， 我们在短时间内会保持两倍的内存占用量。

另外一种解决方案是把 map 的值调整为指针引用： `map[int]*[128]byte` 。 这个方案解决不了有大量存储桶的问题， 但是每个桶的值占用的内存量只需要考虑指针大小， 只需要4个字节而不是128个字节(64位操作系统是8个字节，32位操作系统是4个字节)。

假如带入上面提到的场景，让我们对比一下两种方案的内存消耗情况：

| 步骤            | map[int][128]byte | map[int]\*[128]byte |
|---------------|-------------------|---------------------|
| 分配空 map 字典 | 0 MB              | 0 MB                |
| 添加一百万个元素 | 461 MB            | 182 MB              |
| 删除一百万个元素并进行垃圾回收 | 293 MB            | 38 MB               |

> 如果 map 的键或值的大小超过128个字节，那么 Go 不会直接把它存在存储桶中，而是保存的指针引用。

总结一下，正如我们所看到的，如果我们添加了N个元素然后再删除所有元素，删除之后对应的存储桶的空间不会被释放还会在内存中占用。所以我们在使用的时候需要知道 Go 里的 map 内存占用只会一直增大不会被释放。也没有一个策略可以自动释放缩小 map 的内存占用。如果内存占用比较高的时候，可以通过其他方式比如重建 map 或通过指针来释放或者缓解内存占用问题。


## 是不是内存泄露？ {#是不是内存泄露}

正如reddit 这篇评论所谈论的 [maps_and_memory_leaks_in_go](https://www.reddit.com/r/golang/comments/xq6lm8/maps_and_memory_leaks_in_go/) ，到底是不是“内存泄露”是个问题。大概有以下几个视角：

1.  是优化，重用时不需要分配新的桶；
2.  内存泄露通常被定义为通过某种方式失去了对已分配的内存的引用，从而导致无法释放，在这个场景下清理了 map 仍然能释放内存空间；
3.  数据结构容量的占用，不属于内存泄露的一种；
4.  如果它是有意这么设计的，那么它就不是内存泄露；

我的看法是，在大部分场景下，我们不会像上面这篇文章一样去使用 map ，在多数场景下都是使用后释放掉整个 map；也意味着这样设计的好处，就是在高频写（插入删除）操作的场景下，有更好的性能表现，桶的空间“预分配了”。

在一部分实际使用的场景下，也有不少的问题；比如拓展 Redis 缓存，使用 map 做内存二级缓存，与此同时也有一些提议或反馈，例如支持在删除时释放内存占用、支持 Shrink 释放函数、实现 `SwissTable` 字典表、使用可扩展哈希算法：

-   [Memory so high, and when clean not reduce size](https://github.com/allegro/bigcache/issues/355)
-   [maps do not shrink after elements removal(delete)](https://github.com/patrickmn/go-cache/issues/110)
-   [proposal: x/exp/maps: Shrink](https://github.com/golang/go/issues/54454)
-   [Wikipedia: Extendiable hashing](https://en.wikipedia.org/wiki/Extendible_hashing)
-   [Go Exteniable hashing hash table](https://github.com/rip-create-your-account/finnishtable)

在 Go 中，还处于一个 WIP 的话题。

---

Refs:

-   [runtime: use SwissTable](https://github.com/golang/go/issues/54766)
-   [Memory leak](https://en.wikipedia.org/wiki/Memory_leak)
-   [runtime: make the scavenger more prompt](https://github.com/golang/go/issues/16930)
-   [Go Swisstable](https://github.com/thepudds/swisstable)
