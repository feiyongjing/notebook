### 一个小demo，内部有用户注册、登录、退出登录、查看个人信息，并且设置了cookie、session
~~~go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/google/uuid"
	"net/http"
)

type User struct {
	Username string `json:"username" binding:"required"`
	Password string `json:"password" binding:"required"`
	Age      int    `json:"age"`
}

type Result struct {
	Code string `json:"code"`
	Msg  string `json:"msg"`
	Data any    `json:"data"`
}

func Ok(msg string, data any) Result {
	if msg == "" {
		msg = "一切OK"
	}
	return Result{Code: "000000", Msg: msg, Data: data}
}

func Failed(msg string, data any) Result {
	if msg == "" {
		msg = "系统执行出错"
	}
	return Result{Code: "99999", Msg: msg, Data: data}
}

// 用户列表，可以改成数据库存储
var users map[string]User = make(map[string]User)

// 登录的session和用户信息，可以改成redis缓存存储
var session map[string]User = make(map[string]User)

// 自定义的cookie名称
var myCookie string = "myCookie"

// 默认不检查登录的路由路径
var pathSet map[string]void = make(map[string]void)

// 定义空的结构体，在go中空的结构体是不占用内存空间的
type void struct{}

func init() {
	var member void
	pathSet["/v1/register"] = member
	pathSet["/v1/login"] = member
}

func main() {
	router := gin.Default()

	// 添加中间件拦截器（可以添加多个）,拦截器可以用于判断登录、鉴权、日志等功能
	router.Use(loginCheck)

	v1 := router.Group("/v1")
	{
		v1.POST("/register", register)
		v1.POST("/login", login)
	}

	v2 := router.Group("/v2")
	{
		v2.POST("/logout", logout)
		v2.POST("/userInfo", getUserInfo)
	}

	router.Run(":8080")
}

// 统一判断是否登录
func loginCheck(context *gin.Context) {
	res := Result{}
	fullPath := context.FullPath()

	fmt.Println(fullPath)
	_, ok := pathSet[fullPath]
	if ok {
		// 继续执行下面的中间件拦截器或者是路由回调函数处理
		context.Next()
	} else {
		cookie, err := context.Cookie(myCookie)
		if err == nil {
			_, ok := session[cookie]
			if ok {
				return
			} else {
				res = Ok("用户未登录，请先进行登录", nil)
			}
		} else {
			res = Ok("用户未登录，请先进行登录", nil)
		}
		context.JSON(http.StatusOK, res)
		// 终止处理，并且直接返回响应
		context.Abort()
	}
}

func register(context *gin.Context) {
	res := Result{}
	user := User{}
	if context.ShouldBind(&user) == nil {
		username := user.Username
		_, ok := users[username]
		if ok {
			res = Ok("该用户名已被注册了", nil)
		} else {
			users[username] = user
			res = Ok("注册成功", nil)
		}
	} else {
		res = Failed("非法参数", nil)
	}
	context.JSON(http.StatusOK, res)
}

func login(context *gin.Context) {
	res := Result{}
	user := User{}
	if context.ShouldBind(&user) == nil {
		u, ok := users[user.Username]
		if ok && u.Password == user.Password {
			// uuid生成，需要提前执行 go get github.com/google/uuid 安装库
			uuidValue := uuid.New().String()
			session[uuidValue] = u

			context.SetCookie(myCookie, uuidValue, 3600, "/", "localhost", false, true)
			res = Ok("登录成功", nil)
		} else {
			res = Ok("用户名或密码错误", nil)
		}
	}
	context.JSON(http.StatusOK, res)
}

func logout(context *gin.Context) {
	cookie, _ := context.Cookie(myCookie)
	delete(session, cookie)
	res := Ok("退出登录成功", nil)
	context.JSON(http.StatusOK, res)
}

func getUserInfo(context *gin.Context) {
	cookie, _ := context.Cookie(myCookie)
	user := session[cookie]
	res := Ok("", User{Username: user.Username, Age: user.Age})
	context.JSON(http.StatusOK, res)
}
~~~