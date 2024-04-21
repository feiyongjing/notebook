### go的web框架选择
1. gin是一个golang的web微框架，封装比较优雅，API友好，源码注释比较明确，具有快速灵活，容错方便等特点
2. go原生的net/http也可用于开发web服务

---

# gin使用
### 第一步：依赖引入
~~~shell
# 引入方式一
# 配置文件引入 require github.com/gin-gonic/gin v1.6.3
# 源码文件中添加依赖 import "github.com/gin-gonic/gin" 
# 执行命令进行自动根据源码文件进行下载依赖引入 go mod tidy

# 引入方式二：直接引入并且下载依赖
go get github.com/gin-gonic/gin@v1.9.1
# 后续在源码中可用直接使用 import "github.com/gin-gonic/gin" 引入库
~~~

### 第二步：使用gin进行搭建web服务
- 如果需要将路由写在项目不同目录下的文件中请参考：https://zhuanlan.zhihu.com/p/165633941
~~~go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/gin-gonic/gin/testdata/protoexample"
	"log"
	"net/http"
	"time"
)

type User struct {
	Username string `json:"username" binding:"required"` // 如果需要使用请求参数字段绑定是需要添加标记用于json序列化和反序列化
	Password string `json:"password" binding:"required"`
}

type Result struct {
	Code string                 `json:"code"`
	Msg  string                 `json:"msg"`
	Data map[string]interface{} `json:"data"`
}

func main() {
	//1.创建路由
	// gin.Default()函数使用的是 Logger 和 Recovery 中间件
	// Logger 中间件将日志写入 gin.DefaultWriter，默认是输出到标准输出，也就是gin.DefaultWriter = os.Stdout
	// Recovery 中间件会抓取任何 panic。但是如果有 panic 的话，会写入 500的http状态码，而并不会做其他的处理
	// in.New()不会使用这些默认中间件
	router := gin.Default()
	// 为 multipart forms 设置较低的内存限制 (默认是 32 MiB)，限制上传文件大小为8M
	router.MaxMultipartMemory = 8 << 20

	//2.绑定Restful路由规则，当前请求的路由匹配时会执行对应的函数
	router.GET("/getHello", getHello)

	// 路由组就是设置指定前缀的请求url都被路由到这个路由组中，另外路由组是可以继续在内部嵌套路由组的
	// 路由组: v1
	v1 := router.Group("/v1")
	{
		v1.GET("/user/:name", getPathParam)
		v1.GET("/user", getQuery)
	}

	// 路由组: v2
	v2 := router.Group("/v2")
	{
		v2.GET("/someXML", someXML)
		v2.GET("/someYAML", someYAML)
		v2.GET("/someProtoBuf", someProtoBuf)
	}

	// 路由组: v3
	v3 := router.Group("/v3")
	{
		v3.POST("/login1", getPostForm)
		v3.POST("/login2", getPostFormBind)
		v3.POST("/uploadFile", uploadFile)
		v3.POST("/uploadFiles", uploadFiles)
	}

	// 路由组: v4
	v4 := router.Group("/v4")
	{
		v4.PUT("/xxxput", func(context *gin.Context) {})
		v4.DELETE("/xxxdelete", func(context *gin.Context) {})
	}
	// 路由组: v5
	v5 := router.Group("/v5")
	{
		v5.GET("/longAsync", longAsync)
		v5.GET("/getCookie", getCookie)

		// 重定向，get和post的重定向不同，并且还可以重定向到自身内部服务
		v5.GET("/getMethodRedirect", getMethodRedirect)
		v5.POST("/postMethodRedirect", postMethodRedirect)
		v5.GET("/selfRedirect1", func(c *gin.Context) {
			c.Request.URL.Path = "/v5/selfRedirect2"
			router.HandleContext(c)
		})
		v5.GET("/selfRedirect2", selfRedirect2)

	}

	//3.监听端口，默认本机8080
	router.Run(":8080")
}

func getHello(context *gin.Context) {
	// 设置响应状态码和直接响应的字符串
	//context.String(http.StatusOK, "Hello")

	// 设置响应状态码和响应json
	// gin.H 是 map[string]interface{} 的一种快捷方式
	context.JSON(http.StatusOK, gin.H{
		"message": "Hello",
	})
}

func getPathParam(context *gin.Context) {
	// Param()函数获取path中的路径参数
	name := context.Param("name")
	context.String(http.StatusOK, "你是"+name)
}

func getQuery(context *gin.Context) {
	// Query()函数获取路径后面的key-value参数，如果没有传递也可以使用DefaultQuery()函数设置默认值
	// name := context.DefaultQuery("name", "哈哈")
	name := context.Query("name")
	context.String(http.StatusOK, "你是"+name)
}

func someXML(c *gin.Context) {
	// XML()函数设置返回xml格式的数据
	c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
}

func someYAML(c *gin.Context) {
	// YAML()函数设置返回yaml格式的数据，注意在浏览器中会直接下载这个yaml文件
	c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
}

func someProtoBuf(c *gin.Context) {
	reps := []int64{int64(1), int64(2)}
	label := "test"
	// protobuf 的具体定义写在 testdata/protoexample 文件中。
	data := &protoexample.Test{
		Label: &label,
		Reps:  reps,
	}
	// 请注意，数据在响应中变为二进制数据
	// 将输出被 protoexample.Test protobuf 序列化了的数据
	c.ProtoBuf(http.StatusOK, data)
}

func getPostForm(context *gin.Context) {
	// DefaultPostForm()函数设置默认的表单参数，表单的类型默认是x-www-form-urlencoded或from-data格式
	// PostForm()函数默认解析的是x-www-form-urlencoded或from-data格式的参数进行获取
	types := context.DefaultPostForm("type", "post")
	username := context.PostForm("username")
	password := context.PostForm("password")
	fmt.Printf("username:%s , password:%s , types:%s", username, password, types)
	// JSON()函数可以直接返回结构体，会自动转化为json，需要注意的是结构体的字段需要标记json转化后的字段名称和类型
	context.JSON(http.StatusOK, Result{Code: "000000", Msg: "一切ok"})
}

func getPostFormBind(context *gin.Context) {
	// 可以context.ShouldBindWith使用显式绑定声明绑定那种格式的参数
	// 注意该函数只能绑定一次参数，如果需要多次绑定请修改请求为Post方式，然后使用context.ShouldBindBodyWith函数进行多次绑定请求头
	// 第一个参数是结构体指针，最终数据会存储在这个结构体指针指向的结构体中
	// 第二个参数是绑定的类型，具体的详情如下
	// binding.Header 从HTTP请求头中获取数据并且绑定数据到结构体
	// binding.Query 用于绑定URL路径后面跟上的key-value参数
	// binding.Uri 是当请求使用URL的path传递参数时绑定路径中的数据
	// binding.Form 是当请求的Content-Type是application/x-www-form-urlencoded时使用，用于绑定请求body
	// binding.FormPost 是用于当Post请求的Content-Type是application/x-www-form-urlencoded时使用，用于绑定请求body
	// binding.JSON 是当请求的Content-Type是application/json时使用，用于绑定请求body
	// binding.FormMultipart 是当请求的Content-Type是multipart/form-data时使用，用于绑定请求body，适用于传输文件
	// binding.ProtoBuf 是当请求的Content-Type是application/x-protobuf时使用，用于处理Protobuf编码的body，适用于需要高效数据交换的场合
	// binding.XML 是当请求的Content-Type是application/xml时使用，用于绑定请求body
	// binding.YAML 是当请求的Content-Type是application/x-yaml时使用，用于绑定请求body
	// binding.TOML 是当请求的Content-Type是application/toml时使用，用于绑定请求body，TOML是一种用于配置文件的数据序列化语言，类似于YAML和JSON，但它旨在更明确、更易于阅读和写入
	// binding.MsgPack 是当请求的Content-Type是application/msgpack时使用，用于绑定请求body，MessagePack 是一个高效的二进制序列化格式，类似于JSON，但更小、更快、更节省空间
	// 注意上述不同的类型绑定需要在结构体字段中添加不同的标记
	// 例如 binding.Header绑定需要添加header标记、binding.Query绑定需要添加query标记、
	// binding.Form绑定需要添加form标记、binding.JSON绑定需要添加json标记、binding.Uri绑定需要添加url标记等以此类推
	// context.ShouldBindWith(&user, binding.Form)

	// 如果是想要多次绑定请求体请使用context.ShouldBindBodyWith，然后第二个参数只能设置为请求体类型Content-Type对应的binding
	// context.ShouldBindBodyWith(&user, binding.JSON)

	// 如果是只绑定一次也可以简单地使用 ShouldBind 方法或者 ShouldBindXXX() 方法进行自动绑定
	// 注意结构体字段需要提前添加标记用于json序列化和反序列化
	user := User{}
	// 在这种情况下，将自动选择合适的绑定
	if context.ShouldBind(&user) == nil {
		if user.Username == "哈哈" && user.Password == "password" {
			context.JSON(200, gin.H{"msg": "登录成功"})
		} else {
			context.JSON(401, gin.H{"msg": "用户名或密码错误"})
		}
		return
	}
	context.String(http.StatusOK, fmt.Sprintf("非法错误"))
}

func uploadFile(context *gin.Context) {
	// FormFile()函数读取multipart/form-data类型并且指定字段表示的文件
	file, err := context.FormFile("myInputFile")
	if err != nil {
		context.String(500, "上传图片出错")
	}
	// SaveUploadedFile()函数用于将文件保持到指定的目录中
	context.SaveUploadedFile(file, file.Filename)
	context.String(http.StatusOK, file.Filename+"文件上传成功")
}

func uploadFiles(context *gin.Context) {
	// MultipartForm()函数读取所有multipart/form-data类型的字段
	form, err := context.MultipartForm()
	if err != nil {
		context.String(http.StatusBadRequest, fmt.Sprintf("get err %s", err.Error()))
	}
	// 获取指定字段的所有文件
	files := form.File["myFiles"]
	// 遍历所有文件并且保存到指定的目录下
	for _, file := range files {
		if err := context.SaveUploadedFile(file, file.Filename); err != nil {
			context.String(http.StatusBadRequest, fmt.Sprintf("upload err %s", err.Error()))
			return
		}
	}
	context.String(200, fmt.Sprintf("upload ok %d MyFiles", len(files)))
}

func longAsync(c *gin.Context) {
	// gin.Context由于不能在新的协程中使用需要，创建拷贝gin.Context的副本在新的协程中使用
	cCp := c.Copy()
	go func() {
		// 用 time.Sleep() 模拟一个长任务。
		time.Sleep(5 * time.Second)
		// 请注意您使用的是复制的上下文 "cCp"，这一点很重要
		log.Println("Done! in path " + cCp.Request.URL.Path)
	}()
}

func getCookie(c *gin.Context) {
	// 直接从上下文中获取cookie
	cookie, err := c.Cookie("gin_cookie")
	if err != nil {
		// 直接在上下文中设置cookie
		c.SetCookie("gin_cookie", "test", 3600, "/", "localhost", false, true)
	}
	fmt.Printf("Cookie value: %s \n", cookie)
}

func getMethodRedirect(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.baidu.com/")
}

func postMethodRedirect(c *gin.Context) {
	c.Redirect(http.StatusFound, "http://www.baidu.com/")
}

func selfRedirect2(c *gin.Context) {
	c.JSON(200, gin.H{"hello": "world"})
}
~~~
---

## 请求参数合法验证
~~~go
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/go-playground/validator/v10"
	"reflect"
)

type UserInfo struct {
	Username string `json:"username" binding:"required" msg:"用户名不能为空"` //  通过结构体字段的binding标记进行验证字段的合法性
	Password string `json:"password" binding:"min=3,max=6" msg:"密码长度不能小于3大于6"`
	Email    string `json:"email" binding:"email" msg:"邮箱地址格式不正确"`
}

// 获取异常错误，并且获取结构体中的对应的异常报错信息返回
func GetValidMsg(err error, obj any) string {
	// 使用的时候，需要传obj的指针
	getObj := reflect.TypeOf(obj)
	// 将err接口断言为具体类型
	if errs, ok := err.(validator.ValidationErrors); ok {
		// 断言成功
		for _, e := range errs {
			// 循环每一个错误信息
			// 根据报错字段名，获取结构体的具体字段
			if f, exits := getObj.Elem().FieldByName(e.Field()); exits {
				msg := f.Tag.Get("msg")
				return msg
			}
		}
	}

	return err.Error()
}

func getUser(c *gin.Context) {
	var userInfo UserInfo
	// 请求的参数绑定到对应的结构体，如果有绑定出错就返回错误信息
	err := c.ShouldBindJSON(&userInfo)
	if err != nil {
		fmt.Println(err)
		c.JSON(200, gin.H{"msg": GetValidMsg(err, &userInfo)})
		return
	}
	c.JSON(200, userInfo)
}

func main() {
	router := gin.Default()

	router.POST("/getUser", getUser)
	router.Run(":8000")
}
~~~

### 常见的请求参数验证器
~~~
// 字段不能为空，并且不能没有这个字段
required： 必填字段，如：binding:"required"  

// 针对字符串字段的验证
min 最小长度，如：binding:"min=5"
max 最大长度，如：binding:"max=10"
len 长度，如：binding:"len=6"
contains=fengfeng  // 包含fengfeng的字符串
excludes // 不包含
startswith  // 字符串前缀
endswith  // 字符串后缀

// 针对数字字段的验证
eq 等于，如：binding:"eq=3"
ne 不等于，如：binding:"ne=12"
gt 大于，如：binding:"gt=10"
gte 大于等于，如：binding:"gte=10"
lt 小于，如：binding:"lt=10"
lte 小于等于，如：binding:"lte=10"

// 针对同级字段的
eqfield 等于其他字段的值，如：PassWord string `binding:"eqfield=Password"`
nefield 不等于其他字段的值

// 忽略这个字段的验证
- 忽略字段，如：binding:"-"

// 常量验证，只能是指定的常量，例如如下的red 或green
oneof=red green 

// 数组
dive  // dive后面的验证就是针对数组中的每一个元素

// 日期验证  1月2号下午3点4分5秒在2006年
datetime=2006-01-02
~~~


## 高级用法
1. 自定义拦截器
2. 日志输出到文件

### 自定义拦截器
~~~go
package main

import (
	"github.com/gin-gonic/gin"
)


func main() {
    // gin.Default()函数使用的是 Logger 和 Recovery 中间件
	// gin.New()新建一个没有任何默认中间件的路由
	r := gin.New()

	// 全局中间件
	// Logger 中间件将日志写入 gin.DefaultWriter，即使你将 GIN_MODE 设置为 release。
	// By default gin.DefaultWriter = os.Stdout
	r.Use(gin.Logger())

	// Recovery 中间件会 recover 任何 panic。如果有 panic 的话，会写入 500。
	r.Use(gin.Recovery())

	// 你可以为每个路由添加任意数量的中间件。
	r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

	// 认证路由组
	// authorized := r.Group("/", AuthRequired())
	// 和使用以下两行代码的效果完全一样:
	authorized := r.Group("/")
	// 路由组中间件! 在此例中，我们在 "authorized" 路由组中使用自定义创建的 
    // AuthRequired() 中间件
	authorized.Use(AuthRequired())
	{
		authorized.POST("/login", loginEndpoint)
		authorized.POST("/submit", submitEndpoint)
		authorized.POST("/read", readEndpoint)

		// 嵌套路由组
		testing := authorized.Group("testing")
		testing.GET("/analytics", analyticsEndpoint)
	}

	// 监听并在 0.0.0.0:8080 上启动服务
	r.Run(":8080")
}
~~~


### 日志输出到文件
~~~go
package main

import (
	"github.com/gin-gonic/gin"
	"io"
	"os"
)

func main() {
    // 禁用控制台颜色，将日志写入文件时不需要控制台颜色。
    gin.DisableConsoleColor()

    // 强制日志颜色化，根据检测到的 TTY，控制台的日志输出默认是有颜色的
    // gin.ForceConsoleColor()

    // 记录到文件。
    f, _ := os.Create("gin.log")
    gin.DefaultWriter = io.MultiWriter(f)

    // 如果需要同时将日志写入文件和控制台，请使用以下代码。
    // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)

    router := gin.New()
    router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
		// 你的自定义日志格式
		return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
				param.ClientIP,
				param.TimeStamp.Format(time.RFC1123),
				param.Method,
				param.Path,
				param.Request.Proto,
				param.StatusCode,
				param.Latency,
				param.Request.UserAgent(),
				param.ErrorMessage,
		)
	}))

    router.Use(gin.Recovery())  // 启用组件

    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })

    router.Run(":8080")
}
~~~

