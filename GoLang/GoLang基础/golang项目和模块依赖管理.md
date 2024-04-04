### go项目的识别文件是go.mod，同时该文件也记录了项目的依赖库
~~~shell
# 通过如下命令生成一个go.mod文件
go mod init [项目名称]
~~~

### go.mod文件
~~~
// 模块名称
module demo1

// go的版本
go 1.22

// 依赖的库
require (
//	[依赖的库名称] [库版本]
)
~~~

### 安装依赖
~~~shell
go get [远程代码库和版本]
~~~