### go项目的识别文件是go.mod，同时该文件也记录了项目的依赖库
~~~shell
# 通过如下命令生成一个go.mod文件
go mod init [项目名称]

# 下载依赖包缓存的本地，是下载到GOPATH变量路径下的pkg/mod，通过go env 查看GOPATH
# 注意download只会下载当前的包，而不会下载它所依赖的包
# 在 https://pkg.go.dev/ 搜索查找需要依赖
go mod download [依赖包路径@版本]

# 下载依赖包缓存的本地，是下载到GOPATH变量路径下的pkg/mod，通过go env 查看GOPATH
# 注意 go get 下载依赖包时会修改go.mod文件的
# 与go mod download不同的是它会下载传递性依赖包
# go get 是对 go mod 项目，添加，更新，删除 go.mod 文件的依赖项(仅源码)。不执行编译。侧重应用依赖项管理
go get [参数] [依赖包路径@版本]
# -u 参数更新升级go.mod中的依赖
# -a 参数标示下载好后直接做 go install

# 根据项目源码实际使用到的包去下载项目缺失的依赖（传递性依赖也会下载），去除项目不使用的依赖
go mod tidy

# 修改go.mod文件，本质和手动修改go.mod文件没有区别
go mod edit [-require | -exclude | -replace | -retract]=[库版本或当前项目的历史版本]

# 查看依赖包的最短依赖链路，即那些依赖包引入了指定依赖包
go mod why [依赖包和版本]

# 下载插件
# go install 会在操作系统会在 GOPATH 中安装 Go 生态的第三方命令行应用。不更改项目 go.mod 文件。侧重可执行文件的编译和安装
go install [插件地址和版本]

# 清除项目在本地缓存依赖包，即清除GOPATH变量路径下的pkg/mod下的依赖包
go clean -modchche
~~~

### go.mod文件
~~~
// 模块名称
module demo1

// go的版本
go 1.22

// 依赖的库
require (
//	[库名称] [库版本]
)

// 排除的库
exclude (
//	[库名称] [库版本]
)

// 替换库，场景是原来的库无法访问、原来的库发生迁移、使用本地二次修改的包替换原来的包
replace (
	[原来的库] [库版本] => [新的库] [库版本]
)

// 撤回当前项目的指定版本，即有些版本有问题就撤回这些版本
retract {
    [当前项目需要撤回的版本]
}
~~~

### 安装依赖
~~~shell
go get [远程代码库和版本]
~~~