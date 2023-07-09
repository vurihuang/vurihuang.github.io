---
layout: post
title: Golang net/http 源码阅读
slug: golang-net-http-server
---

<!-- vim-markdown-toc GFM -->

* [使用 net/http 启动一个 web server](#使用-nethttp-启动一个-web-server)
* [How](#how)

<!-- vim-markdown-toc -->

## 使用 net/http 启动一个 web server

``` go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/hello", hello)

	if err := http.ListenAndServe(":8012", nil); err != nil {
		panic(err)
	}
}

func hello(w http.ResponseWriter, req *http.Request) {
	_, _ = fmt.Fprintf(w, "hello\n")
}
```

## How

我们先来看下 `http.ListenAndServe` 函数：

``` go
func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}
```

会进一步调用 `Server` 的 `ListenAndServer` 函数：

``` go
func (srv *Server) ListenAndServe() error {
    // ...

    addr := srv.Addr
    if addr == "" {
        addr = ":http"
    }
    ln, err := net.Listen("tcp", addr)
    if err != nil {
        return err
    }
    return srv.Serve(ln)
}
```

该函数用来监听指定的 TCP 地址，`Listen` 函数会返回一个 `Listener` 并调用 `Serve` 函数。

``` go
func (srv *Server) Serve(l net.Listener) error {
    // ...

    baseCtx := context.Background()

    // ...

    ctx := context.WithValue(baseCtx, ServerContextKey, srv)
    for {
        rw, err := l.Accept()
        
        // ...

        c := srv.newConn(rw)
        // ...
        go c.serve(connCtx)
    }
}
```

我们先忽略 `Serve` 函数具体实现的细节，先看下大致的函数处理流程。`Serve` 函数接收并为每一个连接请求都创建一个 goroutine 进行处理，`serve` 函数会读取请求的 `request` 并调用 `srv.Handler` 来具体处理对应的请求。

在这里调用了 `l.Accept()` 函数返回一个 `conn` 连接，我们回过头来看下前面 `ln, err := net.Listen("tcp", addr)` 做了些什么事情，`ln` 是 `Listener` 接口的一个实现：

``` go
type Listener interface {
    // Accept 等待并返回请求的连接。
    Accept() (Conn, error)

    // Close 关闭该 listener。
    Close() error

    // Addr 返回该 listener 监听的地址。
    Addr() Addr
}
```

`Listen` 函数这里是直接调用了 `lc.Listen` 函数：

``` go
func Listen(network, address string) (Listener, error) {
    var lc ListenConfig
    return lc.Listen(context.Background(), network, address)
})
```

``` go
// Listen 监听本地网络地址。
// network 的值必须是 "tcp", "tcp4", "tcp6", "unix" 或者 "unixpacket"。
// 如果 address 参数中的端口值为 "" 或者 "0"，例如 "127.0.0.1:" 或者 "[::1]:0"，
// 那么会随机选一个可使用的端口作为使用。
func (lc *ListenConfig) Listen(ctx context.Context, network, address string) (Listener, error) {
    addrs, err := DefaultResolver.resolveAddrList(ctx, "listen", network, address, nil)
    // ...

    var l Listener
    la := addrs.first(isIPv4)
    switch la := la.(type) {
        case *TCPAddr:
            l, err = sl.listenTCP(ctx, la)
        case *UnixAddr:
            l, err = sl.listenUnix(ctx, la)
        // ...
    }

    // ...
    return  l, nil
}
```

前面得到的 `ln Listener ` 实例就是从这里得到，因为我们指定了 `tcp` 参数，这里调用的正是 `sl.listenTCP` 函数。

``` go
func (sl *sysListener) listenTCP(ctx context.Context, laddr *TCPAddr) (*TCPListener, error) {
    // 创建一个 socket，得到 file descriptor 文件描述符，
    fd, err := internetSocket(ctx, sl.network, laddr, nil, syscall.SOCK_STREAM, 0, "listen", sl.ListenConfig.Control)
    // ...
    return &TCPListener{fd: fd, lc: sl.ListenConfig}, nil
}
```

既然我们知道了上面的 `Listener` 指的就是 `TCPListener`，那么上面 `l.Accept()` 函数得到的 `rw` 值又是什么东西呢，
这就还得看下 `TCPListener.Accept` 函数里头返回的具体是什么：

``` go

// Accept 被调用后返回一个连接.
func (l *TCPListener) Accept() (Conn, error) {
    // ...
    c, err := l.accept()
    // ...
    return c, nil
}

func (ln *TCPListener) accept() (*TCPConn, error) {
    fd, err := ln.fd.accept()
    // ...
    tc := newTCPConn(fd)
    // ...
    return tc, nil
}
```

`accept` 函数这里返回的就是一个 TCP 连接对象，所以到目前为止的整体流程是：

1. 首先根据给定协议和地址（地址包含端口号），创建 socket，得到一个 Listener，用来监听特定网络地址的请求；
2. 在一个循环体里不停接收监听地址的请求，处理该 TCP 连接请求；
3. 最终每一个请求都会 `go c.serve(connCtx)` 发起一个 goroutine 来进行处理；

``` go
func (c *conn) serve(ctx context.Context) {
    // ...

    for {
        // 读取 HTTP 请求并解析，将一部分数据填充到 http.Request 对象中。
        w, err := c.readRequest(ctx)
        // ...
        // 进行路由匹配选择对应的 Handler 方法进行处理。
        serverHandler{c.server}.ServeHTTP(w, w.req)
        // ...
        // 收尾工作，write 我们的 response 数据，复用 bufio.Reader 来读取下一次的 request body。
        w.finishRequest()
        // ...
    }
}
```

``` go
func (sh sererHandler) ServeHTTP(rw ResponseWriter, req *Request) {
    handler := sh.srv.Handler
    if handler == nil {
        handler = DefaultServeMux
    }
    if req.RequestURI == "*" && req.Method == "OPTIONS" {
        handler = globalOptionsHandler{}
            }
    handler.ServeHTTP(rw, req)
}
```

还记得我们在一开始调用 `http.HandleFunc()` 函数吗，正是这里将我们自己编写的 handler 添加到 `DefaultServeMux` 中：

``` go
var DefaultServeMux = &defaultServeMux

var defaultServeMux = ServeMux
```

可以看到，在调用 `ListenAndServe` 函数 `http.Handler` 参数为 `nil` 的情况，使用的是 `DefaultServeMux`，用的正是 `ServeMux` 对象：

```go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // 根据路由长度排序的数组，路由长度从最长到最短。
	hosts bool       // 是否存在路由包含主机名，有的话在匹配是必须 host+path 都满足 pattern 才行。
}

type muxEntry struct {
	h       Handler
	pattern string
}
```

我们来看下 handler 是如何添加到我们的 `ServeMux` 中的：

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()

    // ...
    if mux.m = nil {
        mux.m = make(map[string]muxEntry)
    }
    e := muxEntry{h: handler, pattern: pattern}
    mux.m[pattern] = e
    if pattern[len(pattern)-1] == '/' {
        mux.es = appendSorted(mux.es, e)
    }

    if pattern[0] != '/' {
        mux.hosts = true
    }
}

func appendSorted(es []muxEntry, e muxEntry) []muxEntry {
    n := len(es)
    // 得到满足条件的插入下标。
    i := sort.Search(n, func(i int) bool {A
        return len(es[i].pattern) < len(e.pattern)
    })
    if i == n {
        return append(es, e)
    }

    // 先对 slice 进行扩容，再将 pattern 更短的成员放到索引 i 的后面。
    es = append(es, muxEntry{})
    copy(es[i+1:], es[i:])
    es[i] = e
    return es
}
```

知道如何构造 `ServeMux` 后，剩下的就是在得到一个请求，如何根据请求的 path 得到 pattern 对应的 handler 的逻辑了：

``` go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    // ...
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}

func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {
    // ...
    host := stripHostPort(r.Host)
    path := cleanPath(r.URL.Path)

    // 如果 path 是 /tree 并且 handler 没有注册该 pattern，
    // 则尝试重定向到 /tree。
    if u, ok := mux.redirectToPathSlash(host, path, r.URL); ok {
        return RedirectHandler(u.String(), StatusMovedPermanently), u.Path
    }

    if path != r.URL.Path {
        _, pattern = mux.handler(host, path)
        url := *r.URL
        url.Path = path
        return RedirectHandler(url.String(), StatusMovedPermanently), pattern
    }

    return mux.handler(host, r.URL.Path)
}

func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    // ...
    // 如果 pattern 不是 '/' 开头，该值为 true，需要匹配 host+path
    if mux.hosts {
        h, pattern = mux.match(host + path)
    }
    // fallback，再尝试一次
    if h == nil {
        h, pattern = mux.match(path)
    }
    if h == nil {
        h, pattern = NotFoundHandler(), ""
    }

    return
}

// 真正处理路由匹配的业务逻辑。
func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    // 先进行全匹配。
    v, ok := mux.m[path]
    if ok {
        return v.h, v.pattern
    }

    // 根据最左最长优先匹配原则来匹配路由。
    // 如果我们定义的 pattern 为 /hello/，
    // 那么是可以匹配 /hello/, /hello/abc 路由的。
    for _, e := range mux.es {
        if strings.HasPrefix(path, e.pattern) {
            return e.h, e.pattern
        }
    }
    return nil, ""
}
```
