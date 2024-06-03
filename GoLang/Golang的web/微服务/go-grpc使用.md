### gRPC的优势
1. 实现的进程间通信方式高效。gRPC 不使用 JSON 或 XML 等文本格式，而是使用基于二进制协议的protocol buffer与 gRPC 服务、客户端进行通信。此外，gRPC 是在 HTTP/2 之上实现的protocol buffer，这使得进程间通信更快。
2. 具有简单、定义良好的服务接口和协议，gRPC 提供了简单但一致、可靠且可扩展的应用程序开发体验。
3. 强类型。protocol buffer清楚地定义了应用程序之间通信的数据类型，这使得分布式应用程序开发更加稳定。因为静态类型有助于减少你在构建跨多个团队和技术的云原生应用程序时遇到的大多数运行时和交互错误。
4. 支持多语言。gRPC被设计成支持多种编程语言。使用protocol buffer的服务定义与语言无关。因此，你可以选择grpc支持的任意语言，并与任何现有的 gRPC 服务或客户端进行通信。
5. 支持双向流式传输。gRPC 对客户端或服务器端流式传输具有原生支持，这使得开发流媒体服务或流媒体客户端变得更加容易。
6. 内置多种高级特性。gRPC 提供对高级特性的内置支持，例如身份验证、加密、元数据交换、压缩、负载平衡、服务发现等。

### gRPC的缺点
1. gRPC不适合面向外部的服务。当你想将应用程序或服务提供给外部客户端使用时，gRPC 可能不是最合适的协议，因为大多数外部使用者对 gRPC 还很陌生。而且，gRPC 服务的协议驱动、强类型化特性可能会降低你向外部提供的服务的灵活性，因为外部使用者可以控制的东西要少得多。
2. 生态系统相对较小。与传统的 REST/HTTP 协议相比，gRPC 生态系统仍然相对较小。浏览器和移动应用程序对 gRPC 的支持仍处于初级阶段。

### 安装
~~~shell
# 安装protoc命令插件，用来解析.proto文件生成go语言的源码，当然其他编程语音也有对应的命令插件可以根据.proto文件生成对应编程语音的源码
# 安装protoc参考：https://blog.csdn.net/weixin_44299027/article/details/138313465#_2
# protoc下载地址：https://github.com/protocolbuffers/protobuf/releases

# protoc-gen-go插件用于生成 .pb.go代码文件 用于填充、序列化和检索消息类型的代码
go install google.golang.org/protobuf/cmd/protoc-gen-go

# protoc-gen-go-grpc插件用于生成 _grpc.pb.go代码文件是生成的客户端和服务端代码
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc

# 生成go代码例子
protoc --go_out=. --go-grpc_out=. ./users/pkg/myservice/myservice.proto

# 生成代码
protoc [参数] [.proto文件]
# --proto_path 或 -I 参数用途：指定 .proto 文件的搜索路径（可以指定多个路径）
# 示例：protoc --proto_path=src --proto_path=include your_file.proto

# --descriptor_set_out参数用途：将编译结果输出为一个 FileDescriptorSet 二进制文件
# 示例：protoc --descriptor_set_out=output.desc your_file.proto

# --include_imports参数用途：在生成的 FileDescriptorSet 中包含导入的 .proto 文件
# 示例：protoc --descriptor_set_out=output.desc --include_imports your_file.proto

# --include_source_info参数用途：在生成的 FileDescriptorSet 中包含源代码信息
# 示例：protoc --descriptor_set_out=output.desc --include_source_info your_file.proto

# --cpp_out参数用途：生成 C++ 代码
# 示例：protoc --cpp_out=out_dir your_file.proto

# --java_out参数用途：生成 Java 代码
# 示例：protoc --java_out=out_dir your_file.proto

# --python_out参数用途：生成 Python 代码
# 示例：protoc --python_out=out_dir your_file.proto

# --go_out参数用途：生成 .pb.go 代码文件的路径，注意最终文件的路径是这个路径 + .proto 文件中编写的option go_package目录下
# --go-grpc_out参数用途：通过protoc-gen-go-grpc 插件生成 _grpc.pb.go代码文件，注意最终文件的路径是这个路径 + .proto 文件中编写的option go_package目录下
# .pb.go代码文件 用于填充、序列化和检索消息类型的代码
# _grpc.pb.go代码文件是生成的客户端和服务端代码
# 示例：protoc --go_out=out_dir --go-grpc_out=out_dir your_file.proto

# 安装protoc生成代码需要的依赖库
go get google.golang.org/grpc@latest
~~~

### proto文件
~~~proto
syntax = "proto3";
package myservice;

// option 指定输出目录文件
option go_package = "users/pkg/myservice";

message Message{
  string id = 1;
  int64 num =2;
  string age = 3;
}

// service 是关键字
// MyService 服务名称是对应接口的名称
// service服务会对应.pb.go文件里interface,里面的rpc对应接口中的函数
service MyService{
  rpc Echo (Message) returns (Message){}
}
~~~
---

### grpc服务端
~~~go
package main

import (
	"context"
	"fmt"
	"go-kit-demo/users/pkg/myservice"  // 注意这个是生成的_grpc.pb.go代码文件
	"google.golang.org/grpc"
	"net"
)

func main() {
	listen, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Printf("监听端口错误：%+v", err)
		return
	}
	// 创建和注册gRPC服务
	s := grpc.NewServer()
	myservice.RegisterMyServiceServer(s, &server{})

	// 在监听的端口上提供gRPC服务
	err = s.Serve(listen)
	if err != nil {
		fmt.Printf("%+v", err)
		return
	}
}

// 实现服务接口
type server struct {
	myservice.UnimplementedMyServiceServer
}

func (server) Echo(c context.Context, in *myservice.Message) (*myservice.Message, error) {
	fmt.Printf("消息是：%+v", in)
	return in, nil
}
~~~
---

## gRPC-Gateway
gRPC-Gateway 是 Google protocol buffers compiler protoc 的插件。 它读取 protobuf service 定义并生成反向代理服务器( reverse-proxy server) ，该服务器将 RESTful HTTP API 转换为 gRPC。 该服务器是根据服务定义中的 google.api.http 批注(annotations)生成的。

这有助于你同时提供 gRPC 和 HTTP/JSON 格式的 API
### 安装gRPC-Gateway
gRPC-Gateway使用参考：https://www.cnblogs.com/hacker-linner/p/14618862.html
~~~shell
# 安装grpc-gateway网关插件依赖，注意protoc-gen-go和protoc-gen-go-grpc需要提前安装
# go install google.golang.org/protobuf/cmd/protoc-gen-go
# go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway

# 修改原来的.proto文件，引入google/api/annotations.proto，并且在grpc接口上添加 HTTP->gRPC 映射注解，简要例子如下，下面有更加详细的例子
# rpc SayHello (HelloRequest) returns (HelloReply) {
#     option (google.api.http) = {
#       get: "/v1/greeter/sayhello"
#       body: "*"
#     };
# }

# 将以下目录下的文件放入到自己的gRPC-Gateway项目下的google/api下，该目录下应该有annotations.proto文件和http.proto文件
${GOPATH}\pkg\mod\github.com\grpc-ecosystem\grpc-gateway@v1.16.0\third_party\googleapis\google\api

# 执行如下命令生成代码
protoc --proto_path=. --go_out=. --go-grpc_out=. --grpc-gateway_out=. ./users/pkg/myservice/myservice.proto
# --proto_path指定所有的 .proto文件路径，默认会在这些路径的下的google/api中查找上述复制的annotations.proto文件和http.proto文件
# --go_out参数指定protoc-gen-go插件用于生成 .pb.go代码文件 用于填充、序列化和检索消息类型的代码存放的路径
# --go-grpc_out参数指定protoc-gen-go-grpc插件用于生成 _grpc.pb.go代码文件是生成的客户端和服务端代码存放的路径
# --grpc-gateway_out参数指定grpc-gateway网关插件生成 .pb.gw.go代码文件存放的路径，用于启动gateway HTTP 服务将HTTP转换为grpc并且路由到grpc服务
~~~
---

### 修改.proto文件，引入google/api/annotations.proto，在grpc接口上添加 HTTP->gRPC 映射注解
~~~
syntax = "proto3";
package myservice;

// 引入google/api/annotations.proto
import "google/api/annotations.proto";

// option 指定输出目录文件
option go_package = "users/pkg/myservice";

message Message{
  string id = 1;
  int64 num =2;
  string age = 3;
}

// service 是关键字
// MyService 服务名称是对应接口的名称
// service服务会对应.pb.go文件里interface,里面的rpc对应接口中的函数
service MyService{
  rpc Echo (Message) returns (Message){
    // 设置HTTP -> gRPC 映射注解
    option (google.api.http) = {
      post: "/v1/example/echo"
      body: "*"
    };
  }
}
~~~
---

### 编写grpc服务和网关服务
~~~go
package main

import (
	"context"
	"flag"
	"fmt"
	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
	"go-kit-demo/users/pkg/myservice"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"log"
	"net"
	"net/http"
)

var grpcPort = flag.Int("grpcPort", 50051, "the port to serve on")
var getGatewayPort = flag.Int("getGatewayPort", 8080, "the port to restful serve on")

func main() {
	listen, err := net.Listen("tcp", fmt.Sprintf(":%d", *grpcPort))
	if err != nil {
		fmt.Printf("监听端口错误：%+v", err)
		return
	}
	// 1. 创建和注册gRPC服务
	grpcServers := grpc.NewServer()
	grpcServer := &server{}
	myservice.RegisterMyServiceServer(grpcServers, grpcServer)

	// 在监听的端口上提供gRPC服务
	go func() {
		err = grpcServers.Serve(listen)
		if err != nil {
			fmt.Printf("%+v", err)
			return
		}
	}()

	// 2. 启动 HTTP 服务(其实是启动一个grpc客户端)
	gwmux := runtime.NewServeMux()
	//禁用传输安全，即TLS
	opts := []grpc.DialOption{grpc.WithTransportCredentials(insecure.NewCredentials())}
	// 注册grpc服务
	err = myservice.RegisterMyServiceHandlerFromEndpoint(context.Background(), gwmux,
		fmt.Sprintf("localhost:%d", *grpcPort), opts)

	if err != nil {
		log.Fatalln("Failed to register gateway:", err)
	}

	gwServer := &http.Server{
		Addr:    fmt.Sprintf(":%d", *getGatewayPort),
		Handler: gwmux,
	}

	log.Println("Serving gRPC-Gateway on http://0.0.0.0" + fmt.Sprintf(":%d", *getGatewayPort))
	log.Fatalln(gwServer.ListenAndServe())
}

type server struct {
	myservice.UnimplementedMyServiceServer
}

func (server) Echo(c context.Context, in *myservice.Message) (*myservice.Message, error) {
	fmt.Printf("消息是：%+v", in)
	return in, nil
}
~~~









