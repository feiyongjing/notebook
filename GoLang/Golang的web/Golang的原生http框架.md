### 使用默认的路由处理器处理路由
~~~go
package main

import (
	"fmt"
	"net/http"
)

func hello(res http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(res, "v1 hello world")
}

func main() {

    // 设置路由方式1：设置路由路径和对应的处理回调函数
	http.HandleFunc("/v1", hello)

    // 设置路由方式2：设置路由路径和自定义的处理器，需要实现自定义处理器中的回调函数
	http.Handle("/v2", &MyV2Handler{})

	// 配置监听8080端口，读取、写入和空闲超时，以及请求头的最大大小
    // 更加详细的http server截个图配置见下面的详情
	server := &http.Server{
		Addr: ":8080",
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		IdleTimeout:    15 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	
	// http.ListenAndServe(":8080", nil)
	server.ListenAndServe()
}

type MyV2Handler struct {
	http.Handler
}

func (*MyV2Handler) ServeHTTP(res http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(res, "v2 hello world")
}
~~~

### 使用http请求多路复用器处理路由
- 路由管理：ServeMux维护一个URL模式到处理函数的映射。当接收到HTTP请求时，ServeMux会根据请求的URL路径选择合适的处理函数并调用，以响应该请求。
- 模式匹配：ServeMux根据注册到多路复用器的URL模式来匹配请求的URL。它选择最长的匹配前缀。
~~~go
package main

import (
	"fmt"
	"net/http"
)

func hello(res http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(res, "v1 hello world")
}

func main() {
	// 使用http请求多路复用器处理路由
	route := http.NewServeMux()
	route.HandleFunc("/v1", hello)
	route.Handle("/v2", &MyV2Handler{})

	// 配置监听8080端口，自定义的路由处理器、读取、写入和空闲超时，以及请求头的最大大小
	server := &http.Server{
		Addr:           ":8080",
		Handler:        route,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		IdleTimeout:    15 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	server.ListenAndServe()

	// http.ListenAndServe(":8080", route)
}

type MyV2Handler struct {
	http.Handler
}

func (*MyV2Handler) ServeHTTP(res http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(res, "v2 hello world")
}
~~~

### go http server的详细配置如下
~~~go
type Server struct {
    Addr         string         // TCP地址，格式通常是"host:port"
    Handler      Handler        // 处理所有请求的handler，如果为nil，将使用DefaultServeMux
    TLSConfig    *tls.Config    // 可选的TLS配置，用于启动HTTPS
    ReadTimeout  time.Duration  // 从请求连接中读取的最大持续时间
    WriteTimeout time.Duration  // 写入响应的最大持续时间
    IdleTimeout  time.Duration  // 等待下一个请求的最大持续时间
    MaxHeaderBytes int          // 请求头的最大字节数，默认为1 << 20（即1MB）

    // 可选的TLS参数，如果设置了，将在接受连接时自动设置TCP的keep-alives
    ReadHeaderTimeout time.Duration // 在读取请求的头部时的超时

    // Go 1.8+
    ShutdownTimeout time.Duration // 关闭超时

    // Go 1.9+
    ConnState func(net.Conn, ConnState) // 连接状态改变时的回调函数

    // Go 1.11+
    ConnContext func(ctx context.Context, c net.Conn) context.Context // 允许为每个连接添加自定义context

    // Go 1.13+
    BaseContext func(net.Listener) context.Context // 允许为每个服务生成基础context
    ErrorLog    *log.Logger // 用于记录服务器的错误信息

    // Go 1.14+
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler) // 用于管理HTTP/2或更高版本的协议升级和处理
}
~~~














