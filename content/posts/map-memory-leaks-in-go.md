+++
title = """
  探究 Go map "内存泄露"
  """
author = ["vuri"]
lastmod = 2023-11-24T02:10:25+08:00
draft = true
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [Intro](#intro)
- [【译】Go map 内存泄露](#译-go-map-内存泄露)

</div>
<!--endtoc-->


## Intro {#intro}

起因是看到这篇文章[ Maps-and-memory-leaks-in-go](https://teivah.medium.com/maps-and-memory-leaks-in-go-a85ebe6e7e69) 讲述关于 Go Map 的内存泄露问题, 在 [reddit](https://www.reddit.com/r/golang/comments/xq6lm8/maps_and_memory_leaks_in_go/) 底下也有很多有意思的探讨,我们先简单翻译下原文介绍下背景,再结合 reddit 上的评论一起来分析分析.


## 【译】Go map 内存泄露 {#译-go-map-内存泄露}

{{< figure src="https://raw.githubusercontent.com/vurihuang/images/master/imgs202311211109151.png" >}}

当我们在使用 Go map 的时候,我们需要对 map 扩缩容一些重要的特性有一定的了解.咱们来看一个会造成"内存泄露"的例子:

```go
m := make(map[int][128]byte)
```

`map m` 的每个元素值是一个128字节的数组,我们会进行以下操作:

1.  初始化一个空的 map;
2.  添加一百万个元素;
3.  删除 map 的所有元素,并执行垃圾回收;

每一步执行完成,都会打印一下堆的内存分配大小,下面是一个示例:

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

我们初始了一个 map,添加了一百万个元素,删除了一百万个元素,然后执行了一次垃圾回收.同时执行了 `runtime.KeepAlive` 来保留 map m 的引用确保不会被回收,以下是执行结果:

```text
0 MB   # 当 map 被初始化后
461 MB # 当添加一百万个元素后
293 MB # 当删除一百万个元素后
```

我们观察到了什么?刚开始的时候,堆的大小还是很小的,当添加了一百万个元素的时候一下子增长了非常多.当我们期待删除这所有元素的时候会释放掉堆内存,实际情况不符合我们的预期.即使最后我们执行了一次 GC 垃圾回收来释放这些对象,堆的大小依赖还有293 MB 的空间.内存占用虽然减少了,但是表现上和我们想的不太一样.它的原因是为什么?我们需要先深入了解一下 Go 里面的 map 的机制是什么.

map 字典提供了一个无序的 key-value 键值对集合,所有的 keys 都是不重复的.在 Go 里面,map 一般基于哈希表(hash table)来实现:一个一维数组,数组里的每个元素指向存储桶的指针引用,存储桶的结构可以存放 key-value 的键值对,如下图所示:

{{< figure src="https://raw.githubusercontent.com/vurihuang/images/master/imgs202311212256313.png" >}}

图1: 一个展示第一个存储桶的哈希表示例.

每一个存储桶都是固定8个长度大小的数组.如果把一个元素插入到已经满了的存储桶则会溢出,Go 会创建另外一个存储桶也是固定8个长度大小的数组,然后把元素放进去再链接到前一个存储桶.如下图所示:

{{< figure src="https://raw.githubusercontent.com/vurihuang/images/master/imgs202311212257202.png" >}}

图2: 在存储桶溢出的例子中,Go 分配了一个新的存储桶并链接到上一个存储桶.

在 Go 中,map 实际上指向的是 `runtime.hmap` 结构体的指针.这个结构体包含了众多的字段,其中有个 `B` 字段它表示 map 中桶的个数:

[golang hmap](https://github.com/golang/go/blob/3e67f46d4f7d661504d281bdedbd1432c09bd751/src/runtime/map.go#L117)

```go
type hmap struct {
	// ...
	B uint8 // buckets 桶数量,指数值(log_2),可以承载负载因子*2^B次方的元素集合
	// ...
}
```

当添加一百万个元素时, 因为 `2^18 = 262,144` 个存储桶(262,144\*8 &gt; 100万), 所以 `B` 的值等于18. 当我们删除一百万个元素的时候, `B` 的值是多少?答案还是18.因此 map 还是包含了那么多的存储桶.

归根结底 map 中的存储桶数量不会减少, 因此删除元素时不会影响 map 的存储桶数量, 它只会把存储桶中的槽位置空, map 的存储桶数量只会增长,不会减少.

| 步骤            | map[int][128]byte | map[int]\*[128]byte |
|---------------|-------------------|---------------------|
| 分配空 map 字典 | 0 MB              | 0 MB                |
| 添加一百万个元素 | 461 MB            | 182 MB              |
| 删除一百万个元素并进行垃圾回收 | 293 MB            | 38 MB               |

---

Refs:

-   [runtime: use SwissTable](https://github.com/golang/go/issues/54766)
