# kit开发结构
~~~go
               +-----------+
    Request -->| Transport |--> Endpoint --> Service
               +-----------+
                    ^
                    |
    Response <------+

// 从 Transport 组件进入，然后被传递到 Endpoint 组件。Endpoint 组件封装了服务操作的输入和输出，通过调用 Service 组件来处理具体的业务逻辑，并将结果返回给 Endpoint 组件。最后，Endpoint 组件将结果返回给 Transport 组件，由 Transport 组件发送回客户端。
~~~


# go-kit 工具快速生成微服务代码
~~~shell
# 安装 GoKit-CLI 工具
go install github.com/kujtimiihoxha/kit

# 快速生成一个微服务
kit new service [服务名称]
# 生成一个users服务，例如 kit new service users
# 目录结构如下，其他需要的请继续使用kit生成或者手动创建
#   user/
#   |---pkg/
#   |------service/
#   |----------service.go

# 手动导入kit库
go get github.com/go-kit/kit
go get github.com/go-kit/kit/transport/http@v0.13.0
go get github.com/gorilla/mux
# 手动创建一个users服务，例如 kit new service users
# 目录结构如下
#   user/
#   |---pkg/
#   |------cmd/
#   |----------main.go
#   |------service/
#   |----------service.go
#   |------endpoint/
#   |----------endpoint.go
#   |------transport/
#   |----------transport.go
~~~

### main.go的程序的启动入口
~~~go
package main

import "go-kit-demo/users/pkg/transport"

func main() {
	transport.RestRun()
}
~~~
---

### transport.go 即Transport层主要负责与传输协议HTTP，GRPC，THRIFT等相关的逻辑
~~~go
package transport

import (
	httptransport "github.com/go-kit/kit/transport/http"
	"github.com/gorilla/mux"
	"go-kit-demo/users/pkg/endpoint"
	"go-kit-demo/users/pkg/service"
	"net/http"
)

// 启动服务监听指定端口
func RestRun() {
	usersService := service.UsersServiceImpl{}
	httpHandler := MakeHttpHandler(usersService)
	http.ListenAndServe(":8080", httpHandler)
}

func MakeHttpHandler(srv service.UsersService) http.Handler {
	router := mux.NewRouter()
	router.Use(commonMiddleware)

	endpoints := endpoint.MakeEndpoints(srv)
	router.Methods("GET").Path("/uppercase").Handler(httptransport.NewServer(
		endpoints.GetUppercase,
		service.DecodeUserRequest,
		service.EncodeResultResponse,
	))

	router.Methods("Post").Path("/lowercase").Handler(httptransport.NewServer(
		endpoints.PostLowercase,
		service.DecodeUserRequest,
		service.EncodeResultResponse,
	))

	router.Methods("GET").Path("/count").Handler(httptransport.NewServer(
		endpoints.GetCount,
		service.DecodeUserRequest,
		service.EncodeResultResponse,
	))

	return router
}

func commonMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Add("Content-Type", "application/json")
		next.ServeHTTP(w, r)
	})
}
~~~
---

### Endpoint层主要负责request／response格式的转换，以及公用拦截器相关的逻辑；Endpoint 是 go-kit 中最重要的组件之一，它用于封装服务操作的输入和输出。通过 Endpoint，开发者可以将一个服务拆分成多个小的可复用的操作，这些操作可以独立地进行测试、扩展和部署
~~~go
package endpoint

import (
	"context"
	"github.com/go-kit/kit/endpoint"
	"go-kit-demo/users/pkg/service"
)

type Endpoints struct {
	GetUppercase  endpoint.Endpoint
	PostLowercase endpoint.Endpoint
	GetCount      endpoint.Endpoint
}

func MakeEndpoints(srv service.UsersService) Endpoints {
	return Endpoints{
		GetUppercase:  MakeGetUppercaseEndpoint(srv),
		PostLowercase: MakePostLowercaseEndpoint(srv),
		GetCount:      MakeGetCountEndpoint(srv),
	}
}

func MakeGetUppercaseEndpoint(srv service.UsersService) endpoint.Endpoint {
	return func(ctx context.Context, request interface{}) (interface{}, error) {
		userRequest := request.(service.UserRequest)
		return srv.GetUppercase(ctx, userRequest)
	}
}

func MakePostLowercaseEndpoint(srv service.UsersService) endpoint.Endpoint {
	return func(ctx context.Context, request interface{}) (interface{}, error) {
		userRequest := request.(service.UserRequest)
		return srv.PostLowercase(ctx, userRequest)
	}
}

func MakeGetCountEndpoint(srv service.UsersService) endpoint.Endpoint {
	return func(ctx context.Context, request interface{}) (interface{}, error) {
		userRequest := request.(service.UserRequest)
		return srv.GetCount(ctx, userRequest)
	}
}
~~~
---

### Service层则专注于业务逻辑，就是我们的业务类、接口等相关信息存放。Service 是业务逻辑的实现，它是 Endpoint 的实现者。Service 负责处理具体的业务逻辑，通过 Endpoint 将其暴露出去供其他服务进行调用
~~~go
package service

import (
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"strings"
)

// 请求体参数结构体
type UserRequest struct {
	Username string `json:"username"`
	Password string `json:"password"`
	Age      string `json:"age"`
}

// 响应体结果结构体
type ResultResponse struct {
	Code    string `json:"code"`
	Message string `json:"message"`
	Data    any    `json:"data"`
}

// 成功响应
func Ok(msg string, data any) ResultResponse {
	if msg == "" {
		msg = "一切OK"
	}
	return ResultResponse{Code: "000000", Message: msg, Data: data}
}

// 失败响应
func Failed(msg string, data any) ResultResponse {
	if msg == "" {
		msg = "系统执行出错"
	}
	return ResultResponse{Code: "99999", Message: msg, Data: data}
}

// 请求体使用编码器解析参数
func DecodeUserRequest(ctx context.Context, r *http.Request) (interface{}, error) {
	var req UserRequest
	err := json.NewDecoder(r.Body).Decode(&req)
	if err != nil {
		return nil, err
	}
	return req, nil
}

// 响应体使用编码器编码返回结果
func EncodeResultResponse(ctx context.Context, w http.ResponseWriter, response interface{}) error {
	return json.NewEncoder(w).Encode(response)
}

// user服务接口
type UsersService interface {
	GetUppercase(ctx context.Context, userRequest UserRequest) (ResultResponse, error)
	PostLowercase(ctx context.Context, userRequest UserRequest) (ResultResponse, error)
	GetCount(ctx context.Context, userRequest UserRequest) (ResultResponse, error)
}

// user服务接口的具体实现
type UsersServiceImpl struct{}

func (UsersServiceImpl) GetUppercase(ctx context.Context, userRequest UserRequest) (ResultResponse, error) {
	return Ok("大写的用户名是："+strings.ToUpper(userRequest.Username), nil), nil
}

func (UsersServiceImpl) PostLowercase(ctx context.Context, userRequest UserRequest) (ResultResponse, error) {
	return Ok("小写的用户名是："+strings.ToLower(userRequest.Username), nil), nil
}

func (UsersServiceImpl) GetCount(ctx context.Context, userRequest UserRequest) (ResultResponse, error) {
	return Ok(fmt.Sprintf("用户名的长度是：%d", strings.Count(userRequest.Username, "")-1), nil), nil
}
~~~
---


