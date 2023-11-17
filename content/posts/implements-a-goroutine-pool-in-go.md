+++
title = "Golang 实现一个协程池 – rulego/fasthttp workerpool 源码介绍"
author = ["vuri"]
lastmod = 2023-11-17T01:25:04+08:00
tags = ["golang"]
draft = false
+++

<div class="ox-hugo-toc toc">

<div class="heading">Table of Contents</div>

- [为什么要使用 goroutine 协程池](#为什么要使用-goroutine-协程池)
- [如何实现一个简易 goroutine 协程池](#如何实现一个简易-goroutine-协程池)

</div>
<!--endtoc-->


## 为什么要使用 goroutine 协程池 {#为什么要使用-goroutine-协程池}

1.  在并发编程时，可以限制 goroutine 的数量，复用资源，提升性能;
2.  保持 CPU 缓存命中率，让 CPU 缓存处于活跃状态;


## 如何实现一个简易 goroutine 协程池 {#如何实现一个简易-goroutine-协程池}

1.  先对我们的目标进行抽象，池化的对象无非是启动、停止、提交任务:
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

2.  生产端: 从 worker 池中获取一个 worker (`wp.getCh()`),并添加任务到任务队列中:
    ```go
           type workerChan struct {
             lastUseTime time.Time
             ch          chan func()
           }

           func (wp *WorkerPool) Submit(fn func()) error {
             ch := wp.getCh()
             if ch == nil {
               return errors.New("no idle workers")
             }
             ch.ch <- fn
             return nil
           }
    ```

3.  消费端: 从任务队列中获取任务并执行:
    ```go
            func (wp *WorkerPool) workerFunc(ch *workerChan) {
              var fn func()
              for fn = range ch.ch {
                if fn == nil {
                  break
                }
                fn()
                // Reset func
                fn = nil
              }
            }
    ```
4.  有了生产和消费端,我们来看下如何真正创建 worker 以及 worker 的任务队列:
    ```go
            type WorkerPool struct {
              // MaxWorkersCount 最大 worker 上限
              MaxWorkersCount int
              // MaxIdleWorkerDuration worker 存活时间
              MaxIdleWorkerDuration time.Duration

              lock         sync.Mutex
              // workersCount 当前的 worker 数量
              workersCount int
              // ready 就绪的 worker 池
              ready          []*workerChan
              workerChanPool sync.Pool
            }

            func (wp *WorkerPool) getCh() *workerChan {
              var ch *workerChan
              createWorker := false

              // 这里操作的是数组,需要上锁保证并发安全
              wp.lock.Lock()
              ready := wp.ready
              n := len(ready) - 1
              if n < 0 { // 没有可运行的 worker 了
                if wp.workersCount < wp.MaxWorkersCount {
                  createWorker = true
                  wp.workersCount++
                }
              } else {
                // 采用 FILO(First In Last Out)先进后出的策略，最先结束的 worker 优先处理接下来的任务
                ch = ready[n]
                ready[n] = nil
                wp.ready = ready[:n]
              }

              wp.lock.Unlock()

              if ch == nil {
                if !createWorker {
                  return nil
                }
                // 实例化一个 worker
                vch := wp.workerChanPool.Get()
                ch = vch.(*workerChan)

                go func() {
                  wp.workerFunc(ch)
                  wp.workerChanPool.Put(vch)
                }()
              }

              return ch
            }
    ```

5.  接下来我们来看下如何对 worker 池进行初始化,也就是我们一开始的 `Start()` 方法:
    ```go
            func (wp *WorkerPool) Start() {
              if wp.stopCh != nil {
                return
              }

              wp.startOnce.Do(func() {
                wp.stopCh = make(chan struct{})
                stopCh := wp.stopCh
                wp.workerChanPool.New = func() any {
                  return &workerChan{
                    ch: make(chan func(), workerChanCap),
                  }
                }

                // TODO: 异步清理 worker
              })
            }

            var workerChanCap = func() int {
              // 当 GOMAXPROCS=1 时,使用阻塞式 chan,
              // 将会立即处理提交的 fn,在 go1.5 以下的版本性能表现会更好.
              if runtime.GOMAXPROCS(0) == 1 {
                return 0
              }

              // 当 GOMAXPROCS>1 的话,使用非阻塞式 chan,
              // 如果 WorkerFunc 是 CPU 绑定(或者说是 CPU 具有亲和性),
              //  worker 任务刚好可以允许被延迟处理
              return 1
            }()
    ```
    我们重点来看下 `workerChanCap` 方法, `runtime.GOMAXPROCS(0)` 什么意思呢,我们来看下注释:

    1.  当我们传入一个参数 `n` 时,会设置 `GOMAXPROCS` 为 `n`,并且返回之前的值;
    2.  而当 `n` &lt;1时又什么都不做,不会修改当前设置值;

    所以其实是一个获取 `GOMAXPROCS` 的小技巧:
    ```go
            // GOMAXPROCS sets the maximum number of CPUs that can be executing
            // simultaneously and returns the previous setting. It defaults to
            // the value of runtime.NumCPU. If n < 1, it does not change the current setting.
            // This call will go away when the scheduler improves.
            func GOMAXPROCS(n int) int {
              if GOARCH == "wasm" && n > 1 {
                n = 1 // WebAssembly has no threads yet, so only one CPU is possible.
              }

              lock(&sched.lock)
              ret := int(gomaxprocs)
              unlock(&sched.lock)
              if n <= 0 || n == ret {
                return ret
              }

              stopTheWorldGC(stwGOMAXPROCS)

              // newprocs will be processed by startTheWorld
              newprocs = int32(n)

              startTheWorldGC()
              return ret
            }
    ```

6.  有了启动的方法,也需要实现清理退出相关的方法,还记得我们在上面 `Start()` 函数预留了一个异步清理的逻辑,以及在退出时的 `Stop()` 逻辑:

    1.  在启动时,同时启动异步清理线程;
    2.  结束时通知并重置所有 worker 进程;
    3.  每个 worker 在运行时检查退出状态(mustStop)决定是否需要继续执行任务,或退出;

    <!--listend-->

    ```go
            func (wp *WorkerPool) Start() {
              // ...

              wp.startOnce.Do(func() {
                // ...

                // 异步清理 worker
                go func() {
                  var scratch []*workerChan
                  for {
                    wp.clean(&scratch)
                    select {
                    case <-stopCh:
                      return
                    default:
                      time.Sleep(wp.getMaxIdleWorkerDuration())
                    }
                  }
                }()
              })
            }

            func (wp *WorkerPool) Stop() {
              if wp.stopCh == nil {
                return
              }
              close(wp.stopCh)
              wp.stopCh = nil

              // 停止所有等待处理任务的 worker
              // 不需要一直等待那些正在处理的 worker 处理完,根据 mustStop 的状态进行判断
              wp.lock.Lock()
              ready := wp.ready
              for i := range ready {
                ready[i].ch <- nil
                ready[i] = nil
              }
              wp.ready = ready[:0]
              wp.mustStop = true
              wp.lock.Lock()
            }

            func (wp *WorkerPool) workerFunc(ch *workerChan) {
              for fn = range ch.ch {
                // ...
                fn = nil

                // 如果进入 mustStop 状态,则直接退出
                if !wp.release(ch) {
                  break
                }
              }

              wp.lock.Lock()
              wp.workersCount--
              wp.lock.Unlock()
            }

            func (wp *WorkerPool) release(ch *workerChan) bool {
              ch.lastUseTime = time.Now()
              wp.lock.Lock()
              if wp.mustStop {
                wp.lock.Unlock()
                return false
              }
              wp.ready = append(wp.ready, ch)
              wp.lock.Unlock()
              return true
            }
    ```
    异步清理任务队列的 `clean()` 代码逻辑:
    ```go
            func (wp *WorkerPool) clean(scratch *[]*workerChan) {
              maxIdleWorkerDuration := wp.getMaxIdleWorkerDuration()
              // 如果 worker 最近的最大存活时间没有处理任务,则进行清理
              criticalTime := time.Now().Add(-maxIdleWorkerDuration)

              wp.lock.Lock()
              ready := wp.ready
              n := len(ready)

              // 通过二分查找出可以被清理的 worker 起始下标
              l, r, mid := 0, n-1, 0
              for l <= r {
                mid = (l + r) / 2
                if criticalTime.After(wp.ready[mid].lastUseTime) {
                  l = mid + 1
                } else {
                  r = mid - 1
                }
              }

              i := r
              if i == -1 {
                wp.lock.Lock()
                return
              }

              *scratch = append((*scratch)[:0], ready[:i+1]...)
              m := copy(ready, ready[i+1:])
              for i = m; i < n; i++ {
                ready[i] = nil
              }
              wp.ready = ready[:m]
              wp.lock.Unlock()

              // 通知 worker 停止退出.
              // 由于任务队列 ch.ch 可能会阻塞,同时也有可能面临 non-local CPUs(即跨核间的并发访问)带来的处理延迟,
              // 这段重置退出逻辑需要放到上锁之外来处理
              tmp := *scratch
              for i := range tmp {
                tmp[i].ch <- nil
                tmp[i] = nil
              }
            }
    ```

把整个代码串起来,就是在 [fasthttp](https://github.com/valyala/fasthttp/blob/master/workerpool.go) 库中的 workerpool 协程池的逻辑,用来高效处理 http connection 连接;
在 [rolego](https://github.com/rulego/rulego/blob/main/pool/workerpool.go) 库中,它进行简单的调整以适配各种 `fn` 函数的任务处理.

---

Refs:

-   [fasthttp workerpool](https://github.com/valyala/fasthttp/blob/master/workerpool.go)
-   [rulego workerpool](https://github.com/rulego/rulego/blob/main/pool/workerpool.go)
-   [ants: a high-performance and low-cost goroutine pool](https://github.com/panjf2000/ants)
