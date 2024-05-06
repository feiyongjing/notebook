### 一个小demo，内部有用户注册、登录、退出登录、查看个人信息，并且设置了cookie、session
~~~go
package main

import (
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

func main() {

	router := gin.Default()
	v1 := router.Group("/v1")
	{
		v1.POST("/register", register)
		v1.POST("/login", login)
		v1.POST("/logout", logout)
		v1.POST("/userInfo", getUserInfo)
	}

	router.Run(":8080")
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
	res := Result{}
	cookie, err := context.Cookie(myCookie)
	if err == nil {
		_, ok := session[cookie]
		if ok {
			delete(session, cookie)
			res = Ok("退出登录成功", nil)
		} else {
			res = Ok("用户未登录，非法退出登录", nil)
		}
	} else {
		res = Ok("用户未登录，非法退出登录", nil)
	}
	context.JSON(http.StatusOK, res)
}

func getUserInfo(context *gin.Context) {
	res := Result{}
	cookie, err := context.Cookie(myCookie)
	if err == nil {
		user, ok := session[cookie]
		if ok {
			res = Ok("", User{Username: user.Username, Age: user.Age})
		} else {
			res = Ok("用户未登录，请先进行登录", nil)
		}
	} else {
		res = Ok("用户未登录，请先进行登录", nil)
	}
	context.JSON(http.StatusOK, res)
}
~~~