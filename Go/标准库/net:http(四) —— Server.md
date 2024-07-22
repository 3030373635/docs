基于 `HTTP` 构建的网络应用包括两个端，即客户端 ( `Client` ) 和服务端 ( `Server` )。

两个端的交互行为包括从客户端发出 `request`、服务端接受 `request` 进行处理并返回 `response` 以及客户端处理 `response`。所以 `http` 服务器的工作就在于如何接受来自客户端的 `request`，并向客户端返回 `response`, 如下图所示：

![img](https://cos.liuqm.cc/v2-bf091a8613d17ddc531ae2b5fda8c472_1440w.webp)

上一章节内容，主要对`net/http`包的`Client`部分进行了介绍，这章节就对 `Server`内容进行介绍。

### HTTP Server简单实现

对于 `golang` 来说，利用 `net/http` 包实现一个`Http Server`非常简单，只需要简简单单几句代码就可以实现，先看看 `Golang` 的其中一种 `http server`简单的实现：

例1：

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "hello World!")
}

func main() {
    //注册路由
    http.HandleFunc("/", handler)
    //创建服务且监听
    http.ListenAndServe(":8080", nil)
}
```

再来看看另外一种`http server`实现，代码如下：

例2：

```go
package main

import (
    "fmt"
    "net/http"
)

type routeIndex struct {
    content string
}

func (route *routeIndex) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, route.content)
}

func main() {
    //注册路由
    http.Handle("/", &routeIndex{content: "Hello, World"})
    //创建服务且监听
    http.ListenAndServe(":8080", nil)
}
```

上述两种方法都实现了简单的 `http server`实现，写法虽然不同，但底层用到的原理其实都是一样的，我们通过源码进行解析。

在上述两种实现种，分别调用了 `http.Handle` 和 `http.HandleFunc` 来实现路由的处理，展开源码看看：

```go
func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }

func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    if handler == nil {
        panic("http: nil handler")
    }
    mux.Handle(pattern, HandlerFunc(handler))
}

func (mux *ServeMux) Handle(pattern string, handler Handler) {
  ....
}
```

总结 `http.Handle` 和 `http.HandleFunc` 函数代码，可以得出以下几个重点内容： `DefaultServeMux` ，`ServeMux`， `Handler` 以及 `ServeMux.Handle`函数，下面逐一展开说明。

### 路由注册

### ServeMux

`ServerMux` 是 `net/http` 包中的一个路由器（`router`），是一个 `HTTP` 请求多路复用器，用于将收到的 `HTTP` 请求路由到相应的处理程序（`handlers`）。

在 `ServerMux` 中，我们可以通过调用 `HandleFunc` 或者 `Handle` 方法来注册一个路由和对应的处理函数。当一个请求到达 `ServerMux` 时，路由器会根据请求的 `URL` 路径找到对应的处理函数，并将请求转发给该函数进行处理。

`ServerMux` 结构体定义如下：

```go
type ServeMux struct {
    mu    sync.RWMutex        //用于保证并发安全性的互斥锁
    m     map[string]muxEntry //一个映射表，将URL模式映射到对应的处理程序。在处理HTTP请求时，ServeMux将使用此映射表来查找与请求URL路径匹配的处理程序
    es    []muxEntry          //一个按长度排序的URL模式条目的切片。这个切片是用来加速ServeMux的URL匹配操作的。在处理HTTP请求时，ServeMux会按照长度递减的顺序迭代这个切片，以便找到最长的匹配URL模式
    hosts bool                //标志位，表示ServeMux是否具有任何带有主机名的URL模式。如果是，则在处理HTTP请求时，ServeMux还需要匹配主机名。如果不是，则可以忽略主机名匹配
}
```

`ServeMux` 是一个非常重要的组件，用于将`HTTP`请求路由到正确的处理程序，并且在`Go`标准库中被广泛使用。在`ServeMux` 中，还有一个 `muxEntry` 结构，`muxEntry`是 `ServeMux` 内部维护的数据结构，用于将 `URL` 路径模式与处理程序相关联。定义代码如下：

```go
type muxEntry struct {
    h       Handler //一个处理程序，它是用于处理与该URL模式匹配的HTTP请求的函数
    pattern string  //与该处理程序相关联的URL模式。在ServeMux中，pattern是映射到处理程序的关键字之一。在匹配请求路径时，ServeMux将使用pattern来判断请求是否与该条目匹配。
}
```

`muxEntry`结构体用于将`URL`模式与处理程序相关联，以便在处理`HTTP`请求时能够正确地路由请求。

再来看看 `DefaultServeMux` ，可以看到 `http.Handle` 和 `http.HandleFunc` 这两个函数最终都由 `DefaultServeMux` 调用 `Handle` 方法来完成路由的注册的，该变量定义如下：

```go
var defaultServeMux ServeMux
var DefaultServeMux = &defaultServeMux
```

这里的 `DefaultServeMux` 表示一个默认的 `ServeMux`，当我们没有创建自定义的 `ServeMux`，则会自动使用一个默认的 `ServeMux`。

### 自定义 ServeMux

我们可以创建自定义的 `ServeMux` 取代默认的 `DefaultServeMux`，示例代码如下：

```go
package main

import (
    "fmt"
    "net/http"
)

type routeIndex struct {
    content string
}

func (route *routeIndex) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, route.content)
}

func htmlHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    html := `<!doctype  html>  
    <META  http-equiv="Content-Type"  content="text/html"  charset="utf-8">    
    <html  lang="zhCN">  
            <head>   
                    <title>Golang</title>  
                    <meta  name="viewport"  content="width=device-width,  initial-scale=1.0,  maximum-scale=1.0,  user-scalable=0;"  />   
            </head>     
            <body>        
            <div  id="app">Hello, HandleFunc World!</div>       
            </body>   
    </html>`
    fmt.Fprintf(w, html)
}

func main() {
    //自定义serveMux
  mux := http.NewServeMux()
    mux.Handle("/Handle", &routeIndex{content: "Hello, Handle World"})
    mux.HandleFunc("/HandleFunc", htmlHandler)
    //创建服务且监听
    http.ListenAndServe(":8080", mux)
}
```

`http.NewServeMux` 方法用于创建一个新的 `ServeMux` 实例，如果调用的是自定义 `ServeMux` 实例 `mux`，那么 `Server` 实例接收到的路由对象将不再是 `DefaultServeMux` 而是 `mux`。

### Handler

了解完 `ServeMux`，再来看看 `Handler`对象：

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

`Handler`对象是个接口，`Handler` 接口中声明了名为 `ServeHTTP` 的函数，也就是说任何结构只要实现了这个 `ServeHTTP` 方法，那么这个结构体就是一个 `Handler` 对象。`http.Handler.ServeHTTP` 方法是用来是用以处理 `request` 并构建 `response` 的核心逻辑所在。

总结起来一句话，**要完成完整的`http server` 服务，必须完成对`Handler`接口的实现，即对 `http.Handler.ServeHTTP` 方法的实现。**

对 `Handler`对象有了大概了解后，回到 `http.Handle` 和 `http.HandleFunc` 函数，看看他们是怎么实现 `Handler` 接口的。

先看`func Handle(pattern string, handler Handler)`函数, 其第二个参数必须为`Handler`接口的实现，所以调用该函数则必须先自行完成对`Handler`接口的实现，方式具体参考示例2。

再看看`func HandleFunc(pattern string, handler func(ResponseWriter, *Request))`函数，发现第二个参数只需传入满足它参数的一个函数即可，并不需要用户自行去实现 `Handler`接口的，究其原因，我们一看源码就明白了：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}

func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    if handler == nil {
        panic("http: nil handler")
    }
    mux.Handle(pattern, HandlerFunc(handler))
}
```

注意一下这行代码：`mux.Handle(pattern, HandlerFunc(handler))`，这里 `HandlerFunc` 实际上是将 `handler` 函数做了一个类型转换，看一下 `HandlerFunc` 的定义：

```go
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

看到这里，应该最终明白了把，原来这里 `HandlerFunc` 实际上已经默认实现了`Handler`接口，实现了它的`ServeHTTP`函数，用户只需要传入 `handler` 函数即可被`HandlerFunc` 强行转为一个`Handler`对象，这是一种巧妙的转换技巧，不需要定义一个结构体，再让这个结构实现 `ServeHTTP` 方法。

### 路由绑定

`ServeMux`内部维护一个`map[string]muxEntry`，该`map`作用将`URL`模式映射到对应的`muxEntry`结构体，而`muxEntry`结构体则将处理程序与`URL`模式相关联。

那`ServeMux`是如何将将`URL`模式映射到对应的`muxEntry`结构体的呢？

通过调用 `http.Handle` 和 `http.HandleFunc` 函数完成映射的。而这两个函数，最终调用的方法的是： `ServeMux.Handle`, 其代码如下：

```go
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    //互斥锁，解决 多个goroutine 并发访问时的线程安全
    mux.mu.Lock()
    defer mux.mu.Unlock()

    //检查参数的合法性，如果有不合法的参数，则会抛出 panic 异常
    if pattern == "" {
        panic("http: invalid pattern")
    }
    if handler == nil {
        panic("http: nil handler")
    }
    if _, exist := mux.m[pattern]; exist {
        panic("http: multiple registrations for " + pattern)
    }
    //初始化ServeMux内部保存URL路径模式和处理程序之间映射关系的map(ServeMux.m)，如果该 map 还未被初始化，则会在此处进行初始化
    if mux.m == nil {
        mux.m = make(map[string]muxEntry)
    }
    //将URL路径模式和处理程序建立映射关系
    //首先，创建一个 muxEntry 结构体，保存处理程序和 URL 路径模式
    //然后，将该 muxEntry 对象加入到 ServeMux 内部维护的 map 中
    e := muxEntry{h: handler, pattern: pattern}
    mux.m[pattern] = e
    //如果URL路径模式的最后一个字符是斜杠（即该 URL 路径模式对应的处理程序是一个目录），则将该 muxEntry 对象插入到 ServeMux.es中
    if pattern[len(pattern)-1] == '/' {
        mux.es = appendSorted(mux.es, e)
    }
    //如果URL路径模式的第一个字符不是斜杠（即该 URL 路径模式对应的处理程序是一个主机名），则将 ServeMux 的 hosts 字段设置为 true
    if pattern[0] != '/' {
        mux.hosts = true
    }
}
```

上述函数主要作用是：

- 将 `URL` 路径模式和处理程序建立映射关系，并将映射关系保存到 `ServeMux` 的内部数据结构中
- 同时，该函数还会对传入的参数进行一些合法性检查，如 `URL` 路径模式和处理程序不能为空，`URL` 路径模式不能重复等
- 最后，该函数还会将 `URL` 路径模式按照长度从长到短排序，并标记 `ServeMux` 是否支持主机名路由。

最后用一张图来总结整个注册路由流程：



![img](https://cos.liuqm.cc/v2-cbf5db648e0796a9aaf465069680efa7_1440w.webp)

ServerMux

### 请求处理

处理完路由相关信息注册，就要进行`TCP`监听服务启动以及`TCP` 连接并处理请求了。标准库提供的 `net/http.ListenAndServe`可以用来监听 `TCP` 连接并处理请求，该函数会使用传入的监听地址和处理器初始化一个 `HTTP` 服务器 `net/http.Server`，调用该服务器的 `net/http.Server.ListenAndServe`方法：

```go
func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}
```

上述代码先创建了一个 `Server` 对象，传入了地址和 `handler` 参数后调用 `Server` 对象 `ListenAndServe()` 方法。

特别需要注意的一点是：该函数的第二个参数是 `Handler` 类型，不管是一个新的 `ServeMux` 对象`mux`，还是默认的 `DefaultServeMux` ， **`ServeMux`其本身自己也实现了 `Handler` 接口，也实现了`ServeHTTP`方法，也是一个 `Handler` 对象，但 `ServeMux` 的 `ServeHTTP()` 方法主要作用是匹配当前路由对应的 `handler` 方法，与自定义的`http.Handler.ServeHTTP` 方法用以处理 `request` 并构建 `response` 功能区别不同**。

下面逐一分析。

### Server

### server结构体

`net/http.Server`是 `HTTP` 服务器的主要结构体，用于控制 `HTTP` 服务器的行为。

其结构体定义为：

```go
type Server struct {
  //服务器监听的地址和端口号，格式为 "host:port"，例如 "127.0.0.1:8080"
    Addr                         string
  //HTTP 请求的处理器。对于每个收到的请求，服务器会将其路由到对应的处理器进行处理。通常使用 http.NewServeMux() 方法创建一个默认的多路复用器，并将其作为处理器。如果没有设置该字段，则使用 DefaultServeMux
    Handler                      Handler
  //一个布尔值，用于指示是否禁用 OPTIONS 方法的默认实现。如果该值为 true，则在收到 OPTIONS 请求时，服务器不会自动返回 Allow 头部，而是交给用户自行处理。默认为 false，即启用 OPTIONS 方法的默认实现
    DisableGeneralOptionsHandler bool
  //HTTPS 服务器的 TLS 配置，用于控制 HTTPS 服务器的加密方式、证书、密钥等安全相关的参数
    TLSConfig                    *tls.Config
  //HTTP 请求的读取超时时间。如果服务器在该时间内没有读取到完整的请求，就会关闭连接。该字段为 time.Duration 类型，默认为 0，表示没有超时限制
    ReadTimeout                  time.Duration
  //HTTP 请求头部读取超时时间。如果服务器在该时间内没有完成头部读取，就会关闭连接。该字段为 time.Duration 类型，默认为 0，表示没有超时限制
    ReadHeaderTimeout            time.Duration
  //HTTP 响应的写入超时时间。如果服务器在该时间内没有完成对响应的写入操作，就会关闭连接。该字段为 time.Duration 类型，默认为 0，表示没有超时限制
    WriteTimeout                 time.Duration
  //HTTP 连接的空闲超时时间。如果服务器在该时间内没有收到客户端的请求，就会关闭连接。该字段为 time.Duration 类型，默认为 0，表示没有超时限制
    IdleTimeout                  time.Duration
  //HTTP 请求头部的最大大小。如果请求头部的大小超过该值，服务器就会关闭连接。该字段为 int 类型，默认为 1 << 20（1MB）
    MaxHeaderBytes               int
    TLSNextProto                 map[string]func(*Server, *tls.Conn, Handler)
  //连接状态变化的回调函数，用于处理连接的打开、关闭等事件
    ConnState                    func(net.Conn, ConnState) 
   //错误日志的输出目标。如果该字段为 nil，则使用 log.New(os.Stderr, "", log.LstdFlags) 创建一个默认的日志输出目标
    ErrorLog                     *log.Logger
  //所有 HTTP 请求的基础上下文。当处理器函数被调用时，会将请求的上下文从基础上下文派生出来。默认为 context.Background()。
    BaseContext                  func(net.Listener) context.Context 
  //连接上下文的回调函数，用于创建连接上下文。每个连接上下文都与一个客户端连接相关联。如果未设置该字段，则每个连接的上下文都是 BaseContext 的副本
    ConnContext                  func(ctx context.Context, c net.Conn) context.Context
  //标志变量，用于表示服务器是否正在关闭。该变量在执行 Shutdown 方法时被设置为 true，用于避免新的连接被接受
    inShutdown                   atomic.Bool 
  //标志变量，用于控制服务器是否支持 HTTP keep-alive。如果该变量为 true，则服务器在每次响应完成后都会关闭连接，即不支持 keep-alive。如果该变量为 false，则服务器会根据请求头部中的 Connection 字段来决定是否支持 keep-alive。该变量在执行 Shutdown 方法时被设置为 true，用于关闭正在进行的
    disableKeepAlives            atomic.Bool
  // 一个 sync.Once 类型的值，用于确保在多线程环境下，NextProtoOnce 方法只被调用一次。NextProtoOnce 方法用于设置 Server.NextProto 字段
    nextProtoOnce                sync.Once 
  // error 类型的值，用于记录 NextProto 方法的调用结果。该值在多个 goroutine 之间共享，用于检测 NextProto 方法是否成功
    nextProtoErr                 error 
  //互斥锁，用于保护 Server 结构体的字段。因为 Server 结构体可能被多个 goroutine 并发访问，所以需要使用互斥锁来确保它们的安全性
    mu                           sync.Mutex    
   //存储 HTTP 或 HTTPS 监听器的列表。每个监听器都是一个 net.Listener 接口类型的实例，用于接收客户端请求。当调用 Server.ListenAndServe() 或 Server.ListenAndServeTLS() 方法时，会为每个监听地址创建一个对应的监听器，并将其添加到该列表中
    listeners                    map[*net.Listener]struct{}
  //表示当前处于活动状态的客户端连接的数量。该字段只是一个计数器，并不保证一定准确。该字段用于判断服务器是否处于繁忙状态，以及是否需要动态调整服务器的工作负载等
    activeConn                   map[*conn]struct{}                                    
  //在服务器关闭时执行的回调函数列表。当服务器调用 Server.Shutdown() 方法时，会依次执行该列表中的每个回调函数，并等待它们全部执行完毕。该字段可以用于在服务器关闭时释放资源、保存数据等操作
    onShutdown                   []func()                                              
  //表示所有监听器的组。该字段包含一个读写互斥锁 sync.RWMutex 和一个映射表 map[interface{}]struct{}。在监听器启动时，会将监听器地址作为键添加到映射表中。该字段主要用于实现优雅地关闭服务器。在服务器关闭时，会遍历所有监听器，逐个关闭它们，并等待所有连接关闭。如果在等待连接关闭时，有新的连接进来，服务器会先将新连接添加到 activeConn 字段中，并等待所有连接关闭后再退出。这样可以保证服务器在关闭过程中，不会丢失任何连接
    listenerGroup                sync.WaitGroup                                        
}
```

### *Server.ListenAndServer

当创建完 一个 `Server` 对象后，调用 `Server` 对象 `ListenAndServe()` 方法会使用网络库提供的 `net.Listen`监听对应地址上的 TCP 连接并通过 `net/http.Server.Serve`处理客户端的请求：

```go
func (srv *Server) ListenAndServe() error {
    //判断服务器是否正在关闭，如果是，则返回
    if srv.shuttingDown() {
        return ErrServerClosed
    }
    //获取服务器监听的地址
    addr := srv.Addr
    //如果服务器监听的地址为空，将其设置为 ":http"
    if addr == "" {
        addr = ":http"
    }
    //创建一个 TCP 监听器，监听指定的地址,如果创建监听器时出现错误，返回错误
    ln, err := net.Listen("tcp", addr)
    if err != nil {
        return err
    }
    //使用 Serve() 方法开始监听并处理连接，返回处理连接时可能出现的错误
    return srv.Serve(ln)
}
```

### *Server.Serve

`net/http.Server.Serve` 方法的主要作用是接收并处理客户端连接，同时调用 `ServeMux` 中的处理程序来处理请求。

代码如下：

```go
func (srv *Server) Serve(l net.Listener) error {
    ...

    // 保证listener只会关闭一次，避免因重复关闭而引起的错误
    origListener := l
    l = &onceCloseListener{Listener: l}
    defer l.Close()

    ...

    // 用于生成handler的基础上下文
    baseCtx := context.Background()
    // 如果设置了BaseContext函数，则使用它生成baseCtx，否则使用默认的background context
    if srv.BaseContext != nil {
        baseCtx = srv.BaseContext(origListener)
        if baseCtx == nil {
            panic("BaseContext returned a nil context")
        }
    }

    var tempDelay time.Duration // 用于Accept失败时的重试时间
    // 用server实例作为value，为handler生成context
    ctx := context.WithValue(baseCtx, ServerContextKey, srv)

    for {
        // 接受连接请求
        rw, err := l.Accept()
        if err != nil {
            // 如果服务器正在关闭，则直接返回错误
            if srv.shuttingDown() {
                return ErrServerClosed
            }
            // 如果连接错误是暂时的，则在等待5毫秒后重试连接，最多重试1秒
            if ne, ok := err.(net.Error); ok && ne.Temporary() {
                if tempDelay == 0 {
                    tempDelay = 5 * time.Millisecond
                } else {
                    tempDelay *= 2
                }
                if max := 1 * time.Second; tempDelay > max {
                    tempDelay = max
                }
                srv.logf("http: Accept error: %v; retrying in %v", err, tempDelay)
                time.Sleep(tempDelay)
                continue
            }
            // 如果是永久性的连接错误，直接返回错误
            return err
        }
        // 根据listener生成的connection context，如果设置了ConnContext函数则使用它生成connCtx，否则使用baseCtx
        connCtx := ctx
        if cc := srv.ConnContext; cc != nil {
            connCtx = cc(connCtx, rw)
            if connCtx == nil {
                panic("ConnContext returned nil")
            }
        }
        tempDelay = 0
        // 根据连接的读写对象，生成新的connection对象
        c := srv.newConn(rw)
        // 设置connection状态
        c.setState(c.rwc, StateNew, runHooks)
        // 启动goroutine，处理连接请求
        go c.serve(connCtx)
    }
}
```

上述代码主要流程为：

1. 创建一个 `onceCloseListener`，将传入的 `net.Listener l` 包装成这个新的 `listener`。`onceCloseListener` 实现了 `net.Listener` 接口，当其 `Close()` 方法被调用时，只会执行一次，避免多次关闭 `listener` 导致的 `panic`。
2. 将 `listener` 包装成 `context.Context` 类型的 `baseCtx`，并赋值给 `ctx`。如果 `srv.BaseContext` 不为空，调用 `srv.BaseContext` 方法将 `listener` 转换成 `context.Context` 类型并赋值给 `baseCtx`。如果 `srv.BaseContext` 返回了 `nil`，直接 `panic` 报错。
3. 进入 for 循环，不断等待新的连接请求。
4. 调用 `l.Accept()` 方法等待新的连接请求，如果出错并且错误是临时性错误，则睡眠一段时间后重试；如果出错并且错误不是临时性错误，则直接返回错误；如果成功接收到一个连接，则将该连接包装成 `context.Context` 类型的 `connCtx`，并将其传给 `srv.ConnContext` 方法处理。如果 `srv.ConnContext` 不为空，调用 `srv.ConnContext` 方法将该连接转换成 `context.Context` 类型并赋值给 `connCtx`。如果 `srv.ConnContext` 返回了 `nil`，直接 `panic` 报错。
5. 调用 `srv.newConn()` 方法创建一个新的 `conn` 实例，将 `listener` 和 `conn` 信息存入 `conn` 实例的字段中。
6. 设置 `conn` 实例的状态为 `StateNew`，同时执行所有的 `hook` 函数。
7. 调用 `conn.serve()` 方法在一个新的 `goroutine` 中处理连接，将 `connCtx` 作为参数传入。如果执行过程中出现 `panic` 错误，则将状态设置为 `StateClosed` 并返回。

`net/http.Server.Serve`函数代码中重点关注分析创建新连接函数：`net/http.Server.newConn` ，设置连接状态函数：`net/http.Conn.setState`和处理连接函数： `net/http.Conn.serve`。

### Conn

### conn结构体

分析`net/http.Server.newConn`函数之前，先来看下 `conn` 结构体的定义：

```go
type conn struct {
    //保存该连接所属的 Server 实例的指针
    server *Server
    //用于取消该连接上下文的函数，用于实现 HTTP 2.0 中取消请求的功能
    cancelCtx context.CancelFunc
    //保存该连接的底层网络连接
    rwc net.Conn
    //连接远程地址
    remoteAddr string
    //如果该连接使用了 TLS，则保存 TLS 的连接状态
    tlsState *tls.ConnectionState
    //最近一次写入错误的错误信息，如果没有则为 nil
    werr error
    //连接的读取器，实现了 io.Reader 接口
    r *connReader
    //用于读取 HTTP 请求的缓冲读取器
    bufr *bufio.Reader
    //用于写入 HTTP 响应的缓冲写入器
    bufw *bufio.Writer
    //最近一次处理的 HTTP 请求方法
    lastMethod string
    //当前正在处理的 HTTP 响应指针
    curReq atomic.Pointer[response]
    //当前连接状态，保存一个 stateAtomic 实例
    curState atomic.Uint64
    mu       sync.Mutex
    //表示该连接是否被劫持
    hijackedv bool
}
```

该结构体的作用是表示一个客户端连接，并提供了 `HTTP` 请求和响应的读写接口。其中，包含了一些关于该连接的元信息，如连接所属的服务器、底层网络连接、远程地址等；以及与 `HTTP` 协议相关的读写缓冲区，以及一些处理并发请求的控制信息，如当前正在处理的请求、当前连接状态等。

了解了此结构体后，回头看看`net/http.Server.newConn`函数：

```go
func (srv *Server) newConn(rwc net.Conn) *conn {
    // 创建 conn 对象，并设置 server 和 rwc 字段
    c := &conn{
        server: srv,
        rwc:    rwc,
    }
    // 如果开启了 debugServerConnections，则为 rwc 字段包装一个 LoggingConn 对象
    if debugServerConnections {
        c.rwc = newLoggingConn("server", c.rwc)
    }
    return c
}
```

该函数主要功能是创建并返回一个包含 `server` 和 `rwc` 字段的 `conn` 对象。

### *conn.setState

该函数主要是用于设置连接的状态，并记录连接的操作历史，以及在状态变化时执行钩子函数。源码如下：

```go
func (c *conn) setState(nc net.Conn, state ConnState, runHook bool) {
    srv := c.server // 获取当前连接的 server 对象
    switch state { 
    // 根据状态值对连接进行相应的操作
    case StateNew:
        srv.trackConn(c, true) // 记录新连接
    case StateHijacked, StateClosed:
        srv.trackConn(c, false) // 移除已经 hijacked 或已经关闭的连接
    }
    // 如果状态值非法，则抛出异常
    if state > 0xff || state < 0 {
        panic("internal error")
    }
    // 将当前状态打包存储到 curState 中，其中高 56 位为时间戳，低 8 位为状态值
    packedState := uint64(time.Now().Unix()<<8) | uint64(state)
    c.curState.Store(packedState)
    // 如果 runHook 为 true，且设置了 srv.ConnState，执行相应的 Hook 函数
    if !runHook {
        return
    }
    if hook := srv.ConnState; hook != nil {
        hook(nc, state)
    }
}
```

### *conn.serve

创建完服务端的连接，设置好连接状态后，就开始创建单独的 `Goroutine` 并在其中调用 `net/http.Conn.serve` 方法，该方法会根据客户端请求中的信息，将请求分发给相应的路由处理器，然后调用相应的路由处理器来处理请求，并生成 `HTTP` 响应。处理完毕后，它会将生成的 `HTTP` 响应发送给客户端，并关闭连接。

源码如下：

```go
func (c *conn) serve(ctx context.Context) {
    //设置远程和本地地址，并将本地地址保存到上下文中
    c.remoteAddr = c.rwc.RemoteAddr().String()
    ctx = context.WithValue(ctx, LocalAddrContextKey, c.rwc.LocalAddr())
    var inFlightResponse *response

    //省略非重点逻辑代码，该段代码为使用 defer 语句设置连接关闭和错误恢复函数。 如果在处理连接期间出现错误，则将连接状态设置为关闭，同时终止悬挂的响应和 HTTP 请求
    defer func() {
        ...
    }()

    //省略非重点逻辑代码，这段代码首先判断这个连接是否是TLS连接，如果是，就进行TLS握手。如果TLS握手失败，记录错误日志并返回
    if tlsConn, ok := c.rwc.(*tls.Conn); ok {
        ...
    }

    //这段代码主要是对 conn 结构体进行初始化

    //设置上下文（使用传入的 ctx 创建一个新的上下文，并获取到可以取消该上下文的函数 cancelCtx）
    ctx, cancelCtx := context.WithCancel(ctx)
    c.cancelCtx = cancelCtx
    defer cancelCtx()
    //创建一个 connReader 结构体作为 conn 的 r 字段（它包装了 conn 的 rwc 字段并添加了读取和超时控制功能）
    c.r = &connReader{conn: c}
    //创建一个 bufio.Reader 作为 conn 的 bufr 字段（用于从连接中读取请求数据，并提供缓存和解析功能）
    c.bufr = newBufioReader(c.r)
    //创建一个 bufio.Writer 作为 conn 的 bufw 字段（用于向连接中写入响应数据，并提供缓存功能）
    c.bufw = newBufioWriterSize(checkConnErrorWriter{c}, 4<<10)

    for {
        //读取请求
        w, err := c.readRequest(ctx)
        //如果剩余可读字节数不等于服务器初始读取限制大小，则设置连接状态为活动状态，并运行挂钩
        if c.r.remain != c.server.initialReadLimitSize() {
            c.setState(c.rwc, StateActive, runHooks)
        }
        //省略非重点逻辑代码，如果读取发生错误，则根据不同类型进行不同处理方案
        if err != nil {
            ...
        }

        //从 http.ResponseWriter 接口的 ResponseWriter 实例中获取请求 Request
        req := w.req
        //如果请求头部包含 Expect: 100-continue，则处理 Expect 请求头部
        if req.expectsContinue() {//如果请求协议为 HTTP/1.1 或更高版本，且请求体长度不为 0，则使用一个新的 expectContinueReader 读取请求体
            if req.ProtoAtLeast(1, 1) && req.ContentLength != 0 {
                req.Body = &expectContinueReader{readCloser: req.Body, resp: w}
                //标记当前请求支持发送 100 Continue 响应
                w.canWriteContinue.Store(true)
            }
        } else if req.Header.get("Expect") != "" {//如果请求头部不包含 Expect: 100-continue，但是包含了 Expect 请求头部，则发送 417 Expectation Failed 响应
            w.sendExpectationFailed()
            return
        }

        //将当前请求存储在连接 c 的 curReq 字段中，表示该连接当前正在处理该请求
        c.curReq.Store(w)

        //如果请求体仍有数据需要读取，则调用 registerOnHitEOF 注册一个回调函数，在读取完请求体后继续处理该请求；否则，立即处理该请求
        if requestBodyRemains(req.Body) {
            registerOnHitEOF(req.Body, w.conn.r.startBackgroundRead)
        } else {
            w.conn.r.startBackgroundRead()
        }

        //将当前请求 w 标记为正在处理中
        inFlightResponse = w
        //使用服务器的处理器处理该请求
        serverHandler{c.server}.ServeHTTP(w, w.req)
        //当前请求处理完毕，将 inFlightResponse 置为 nil
        inFlightResponse = nil
        //取消与该请求关联的上下文
        w.cancelCtx()
        //如果该连接已经被接管（例如 WebSocket），则直接返回
        if c.hijacked() {
            return
        }
        //结束当前请求
        w.finishRequest()
        //取消写超时
        c.rwc.SetWriteDeadline(time.Time{})
        //如果当前请求不支持重用该连接，则关闭该连接
        if !w.shouldReuseConnection() {
            if w.requestBodyLimitHit || w.closedRequestBodyEarly() {
                c.closeWriteAndWait()
            }
            return
        }
        //将该连接状态设置为闲置状态
        c.setState(c.rwc, StateIdle, runHooks)
        //将当前请求信息设置为nil
        c.curReq.Store(nil)
        //如果当前连接所在的服务器不支持 keep-alive，直接返回
        if !w.conn.server.doKeepAlives() {
            return
        }
        //获取当前服务器的空闲超时时间 d，如果 d 不为 0，则将读取截止时间设置为当前时间加上 d，否则将其设置为零时间
        if d := c.server.idleTimeout(); d != 0 {
            c.rwc.SetReadDeadline(time.Now().Add(d))
        } else {
            c.rwc.SetReadDeadline(time.Time{})
        }
        //检查缓冲区中是否有至少 4 字节的未读数据，如果没有则返回
        if _, err := c.bufr.Peek(4); err != nil {
            return
        }
        //将读取截止时间设置为零时间
        c.rwc.SetReadDeadline(time.Time{})
    }
}
```

上述代码比较复杂，细节处理比较多，精简一部分掉仍然存在很多代码，看起来比较头疼。这边总结下核心流程：

1. 通过 `context.WithCancel` 函数创建一个 `context`，用于控制整个请求的生命周期。设置 `c.cancelCtx = cancelCtx`，并在函数结束时执行 `defer cancelCtx()`，以确保在整个请求完成后，`context` 及其所有子 `context` 都被取消。
2. 初始化 `connReader` 和 `bufioReader`，并创建一个 `bufioWriter`。
3. 在一个 `for` 循环中，不断调用`c.readRequest(ctx)`读取客户端发来的请求。如果读取失败，则根据错误类型返回相应的 `HTTP` 响应。
4. 如果读取成功，则根据请求头中的 "Expect" 字段和 "Content-Length" 字段判断是否需要继续等待客户端的数据，并在必要时发送 "100 Continue" 响应。
5. 根据请求的内容，调用`serverHandler{c.server}.ServeHTTP(w, w.req)` 执行相应的处理函数，并在处理完成后决定是否需要继续保持连接或关闭连接。
6. 如果继续保持连接，则在请求处理完成后，等待下一个请求的到来；否则，关闭连接并结束函数执行。
7. 在函数结束时，清理资源，包括关闭连接、取消 `context`、恢复连接状态等。

在这些关键流程中，其中 `c.readRequest` 和 `serverHandler{c.server}.ServeHTTP`这两个函数重点分析。

### *conn.readRequest

函数`net/http.conn.readRequest`是 `conn`结构体中的一个方法，主要作用是从`TCP`连接中读取`HTTP`请求，解析请求行、请求头和请求体，并将其封装成`http.Request`对象和`http.ResponseWriter`对象，返回一个`response`结构体指针。

源码如下：

```go
func (c *conn) readRequest(ctx context.Context) (w *response, err error) {
    //如果连接已经已经被劫持，则直接返回ErrHijacked错误
    if c.hijacked() {
        return nil, ErrHijacked
    }
    //定义超时时间
    //wholeReqDeadline 为整个请求超时时间    hdrDeadline 为读取请求头的超时时间
    var (
        wholeReqDeadline time.Time
        hdrDeadline      time.Time
    )
    //设置相关超时时间，就是在当前时间上加上对应的时间段
    t0 := time.Now()
    if d := c.server.readHeaderTimeout(); d > 0 {
        hdrDeadline = t0.Add(d)
    }
    if d := c.server.ReadTimeout; d > 0 {
        wholeReqDeadline = t0.Add(d)
    }
    c.rwc.SetReadDeadline(hdrDeadline)
    //如果服务端的 WriteTimeout 不为 0，则在函数执行结束时设置写操作的超时时间为当前时间加上WriteTimeout 的时间段
    if d := c.server.WriteTimeout; d > 0 {
        defer func() {
            c.rwc.SetWriteDeadline(time.Now().Add(d))
        }()
    }
    //设置 c.r 的读限制大小，这里设置为默认值 4KB
    c.r.setReadLimit(c.server.initialReadLimitSize())
    //如果上一次请求的方法为 POST，则在 bufio.Reader 中读取并丢弃开头可能存在的 CR 或 LF
    if c.lastMethod == "POST" {
        peek, _ := c.bufr.Peek(4)
        c.bufr.Discard(numLeadingCRorLF(peek))
    }
    //从 bufio.Reader 中读取请求，并解析为 http.Request 结构体
    req, err := readRequest(c.bufr)
    if err != nil {
        //如果读取错误且达到了读限制则返回 errTooLarge
        if c.r.hitReadLimit() {
            return nil, errTooLarge
        }
        return nil, err
    }
    //判断是否为 HTTP/1.x 协议
    if !http1ServerSupportsRequest(req) {
        return nil, statusError{StatusHTTPVersionNotSupported, "unsupported protocol version"}
    }
    //设置上一个请求的方法，解除读限制大小
    c.lastMethod = req.Method
    c.r.setInfiniteReadLimit()
    //从请求头中获取Host字段，haveHost表示Host是否存在
    hosts, haveHost := req.Header["Host"]
    //判断是否使用HTTP/2协议升级
    isH2Upgrade := req.isH2Upgrade()
    //如果请求协议版本>=1.1 且 Host字段不存在且不是使用HTTP/2协议升级，返回缺少必要的Host标头的错误
    if req.ProtoAtLeast(1, 1) && (!haveHost || len(hosts) == 0) && !isH2Upgrade && req.Method != "CONNECT" {
        return nil, badRequestError("missing required Host header")
    }
    //如果Host字段中的值不唯一或者格式不正确，返回Host标头格式不正确的错误
    if len(hosts) == 1 && !httpguts.ValidHostHeader(hosts[0]) {
        return nil, badRequestError("malformed Host header")
    }
    //遍历请求头中的所有字段和值，如果字段名或值不符合HTTP规范，返回相应的错误
    for k, vv := range req.Header {
        if !httpguts.ValidHeaderFieldName(k) {
            return nil, badRequestError("invalid header name")
        }
        for _, v := range vv {
            if !httpguts.ValidHeaderFieldValue(v) {
                return nil, badRequestError("invalid header value")
            }
        }
    }
    //删除请求头中的Host字段
    delete(req.Header, "Host")
    //使用context创建一个新的请求上下文ctx，cancelCtx用于取消请求上下文
    ctx, cancelCtx := context.WithCancel(ctx)
    req.ctx = ctx
    //将请求的远程地址设置为连接的远程地址，TLS设置为连接的TLS状态
    req.RemoteAddr = c.remoteAddr
    req.TLS = c.tlsState
    //如果请求的Body是一个可关闭的body，设置doEarlyClose为true
    if body, ok := req.Body.(*body); ok {
        body.doEarlyClose = true
    }
    //如果读请求头的超时时间和读整个请求的超时时间不相同，将读取连接设置为整个请求的超时时间
    if !hdrDeadline.Equal(wholeReqDeadline) {
        c.rwc.SetReadDeadline(wholeReqDeadline)
    }
    //创建了一个 response 对象 w，并对其进行初始化赋值
    w = &response{
        conn:             c,
        cancelCtx:        cancelCtx,
        req:              req,
        reqBody:          req.Body,
        handlerHeader:    make(Header),
        contentLength:    -1,
        closeNotifyCh:    make(chan bool, 1),
        wants10KeepAlive: req.wantsHttp10KeepAlive(),
        wantsClose:       req.wantsClose(),
    }
    if isH2Upgrade {
        w.closeAfterReply = true
    }
    w.cw.res = w
    //为 w 的消息体写入器 w.w 创建了一个缓冲区，大小为 bufferBeforeChunkingSize。
    w.w = newBufioWriterSize(&w.cw, bufferBeforeChunkingSize)
    //返回 w 和 nil
    return w, nil
}
```

这个函数代码有点繁琐，主要核心流程：

1. 设置读取请求头的超时时间和整个请求的超时时间。
2. 读取请求头的第一行，解析请求头的第一行，获取请求方法、请求`URI`和`HTTP`协议版本。
3. 解析请求头的其他行，获取请求头的键值对；检查请求头是否符合`HTTP`协议的规范，如果不符合，则返回错误。
4. 检查请求头是否包含必需的`Host`字段，如果不包含，则返回错误；检查`Host`字段的格式是否正确，如果格式不正确，则返回错误。
5. 创建一个新的`response`对象，设置该对象的属性；返回`response`对象。

对其代码逻辑进行精简下，主要核心逻辑代码就以下内容：

```go
func (c *conn) readRequest(ctx context.Context) (w *response, err error) {
    //... 定义和设置相关超时时，代码省略
    //从 bufio.Reader 中读取请求，并解析为 http.Request 结构体
    req, err := readRequest(c.bufr)
    // ... 检查各种头部信息是否符合HTTP协议规范
    //使用context创建一个新的请求上下文ctx，cancelCtx用于取消请求上下文
    ctx, cancelCtx := context.WithCancel(ctx)
    req.ctx = ctx
    //将请求的远程地址设置为连接的远程地址，TLS设置为连接的TLS状态
    req.RemoteAddr = c.remoteAddr
    req.TLS = c.tlsState
    ...
    //创建了一个 response 对象 w，并对其进行初始化赋值
    w = &response{
        conn:             c,
        cancelCtx:        cancelCtx,
        req:              req,
        reqBody:          req.Body,
        handlerHeader:    make(Header),
        contentLength:    -1,
        closeNotifyCh:    make(chan bool, 1),
        wants10KeepAlive: req.wantsHttp10KeepAlive(),
        wantsClose:       req.wantsClose(),
    }
    ...
    w.cw.res = w
    //为 w 的消息体写入器 w.w 创建了一个缓冲区，大小为 bufferBeforeChunkingSize。
    w.w = newBufioWriterSize(&w.cw, bufferBeforeChunkingSize)
    //返回 w 和 nil
    return w, nil
}
```

代码中处理读取请求最重要的是逻辑是调用 `readRequest(c.bufr)`， 而`readRequest`函数在`Request`章节部分已经说明过，可以翻阅前面内容参考看看。

### serverHandler.ServeHTTP

在 `net/http.Conn.serve` 方法中循环调用 `readRequest()` 方法读取到一个请求进行处理，其中最关键的逻辑就是一行码：`serverHandler{c.server}.ServeHTTP(w, w.req)`，来看看其相关代码：

```go
type serverHandler struct {
    srv *Server
}

func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
  // 获取处理程序
    handler := sh.srv.Handler
    if handler == nil {
        handler = DefaultServeMux
    }
  // 处理 OPTIONS 请求
    if !sh.srv.DisableGeneralOptionsHandler && req.RequestURI == "*" && req.Method == "OPTIONS" {
        handler = globalOptionsHandler{}
    }
  // 检查 URL 是否包含分号
    if req.URL != nil && strings.Contains(req.URL.RawQuery, ";") {
        var allowQuerySemicolonsInUse atomic.Bool
        req = req.WithContext(context.WithValue(req.Context(), silenceSemWarnContextKey, func() {
            allowQuerySemicolonsInUse.Store(true)
        }))
        defer func() {
            if !allowQuerySemicolonsInUse.Load() {
                sh.srv.logf("http: URL query contains semicolon, which is no longer a supported separator; parts of the query may be stripped when parsed; see golang.org/issue/25192")
            }
        }()
    }
  //调用处理程序的 ServeHTTP 方法处理请求
    handler.ServeHTTP(rw, req)
}
```

总体来说，这个函数的作用是调用适当的处理程序来处理传入的 `HTTP` 请求，并在处理过程中做一些必要的检查。

需要特别注意的是，`sh.srv.Handler` 这个对象并不是我们自定义的实现的`Handler`对象，这个对象是由 `ServeMux`内部实现的。如果我们没有自定义了 `ServeMux`，则使用默认的处理程序 `DefaultServeMux`。其代码定义为：

```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURI == "*" {
        if r.ProtoAtLeast(1, 1) {
            w.Header().Set("Connection", "close")
        }
        w.WriteHeader(StatusBadRequest)
        return
    }
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}
```

上述代码主要核心流程是：

- 通过 `mux.Handler`查找和匹配到相关对应的已注册的路由表达式和 `handler`
- 通过 `h.ServeHTTP`执行`handler`

来看下 `net/http.ServeMux.Handler` 函数以及关联函数，源代码如下：

```go
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {
    // 判断请求方式是否为CONNECT方式
    if r.Method == "CONNECT" {
        // 若是，则重定向到路径以斜杠(/)结尾
        if u, ok := mux.redirectToPathSlash(r.URL.Host, r.URL.Path, r.URL); ok {
            return RedirectHandler(u.String(), StatusMovedPermanently), u.Path
        }
        // 不需要重定向则处理该连接请求
        return mux.handler(r.Host, r.URL.Path)
    }
    // 获取去除端口号的Host
    host := stripHostPort(r.Host)
    // 获取规范化的请求路径
    path := cleanPath(r.URL.Path)
    // 判断请求路径是否需要重定向到以斜杠(/)结尾的路径
    if u, ok := mux.redirectToPathSlash(host, path, r.URL); ok {
        return RedirectHandler(u.String(), StatusMovedPermanently), u.Path
    }
    // 如果请求路径中包含转义字符，则需要重定向到规范化后的路径
    if path != r.URL.Path {
        // 获取能匹配规范化后路径的路由规则，返回的是该路由规则的处理函数和该路由规则
        _, pattern = mux.handler(host, path)
        // 构造重定向到规范化后路径的url，然后返回该url以及匹配到的路由规则
        u := &url.URL{Path: path, RawQuery: r.URL.RawQuery}
        return RedirectHandler(u.String(), StatusMovedPermanently), pattern
    }
    // 如果不需要重定向，则直接获取能匹配原始请求路径的路由规则，返回的是该路由规则的处理函数和该路由规则
    return mux.handler(host, r.URL.Path)
}

func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    mux.mu.RLock()         // 加读锁，防止其他 goroutine 写入 mux
    defer mux.mu.RUnlock() // 函数执行完毕后释放读锁
    if mux.hosts {         // 如果 ServeMux 是按照 host 匹配路由的
        h, pattern = mux.match(host + path) // 尝试匹配 host + path，获取路由处理函数和匹配的路由规则
    }
    // 如果按照 host 匹配路由未成功，继续按照 path 匹配路由
    if h == nil {
        h, pattern = mux.match(path) // 尝试匹配 path，获取路由处理函数和匹配的路由规则
    }
    // 如果按照 path 匹配路由仍然未成功，返回 NotFoundHandler
    if h == nil {
        h, pattern = NotFoundHandler(), ""
    }
    return // 返回匹配的路由处理函数和路由规则
}

func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    // 按照 path 查找是否存在完全匹配的路由
    v, ok := mux.m[path]
    if ok {
        return v.h, v.pattern
    }

    // 依次遍历所有以 / 结尾的路由，尝试寻找一个路由与 path 前缀匹配
    for _, e := range mux.es {
        if strings.HasPrefix(path, e.pattern) {
            return e.h, e.pattern
        }
    }
    // 找不到匹配的路由时返回空
    return nil, ""
}
```

如果请求的路径和路由中的表项匹配成功，我们会调用表项中对应的处理器，处理器中包含的业务逻辑会通过`net/http.ResponseWriter`构建 `HTTP` 请求对应的响应并通过 `TCP` 连接发送回客户端。

最后一张图来总结请求处理的整个流程逻辑：



![img](https://cos.liuqm.cc/v2-a1d03b701e34058f786783bfcf0ced33_1440w.webp)