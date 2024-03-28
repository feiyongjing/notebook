# golang 的编译与运行
- 方式一：先编译再运行，go build [go源码文件] 编译出来的可执行文件（二进制文件，在windows是exe文件）包含了所有需要的库文件，将编译好的二进制文件拿到其他相同的操作系统上直接运行，不需要提前安装go环境，因为所需要的所有资源到打包到二进制文件中了
- 方式二：直接 go run [go源码文件]，这样运行需要提前安装go环境

~~~shell
# 注意 go build [非main包文件目录] 只会编译检查该目录下go文件，不会生成可执行二进制文件

go build [-o 二进制文件名称] [go的main包源码文件，如果有多个文件需要全部指定]

# go的交叉编译，即在当前操作系统编译出另一个操作系统可以运行的二进制文件
# 交叉编译需要修改 GOOS、GOARCH、CGO_ENABLED 三个环境变量，使用go env 可以查看这些环境变量
#   GOOS是设置系统类型
#   GOARCH是设置系统架构
#   CGO_ENABLED是设置是否开启go对c语言模块的支持，即同时可以编译c代码打包运行，但是交叉编译是不支持开启go对c语言模块的支持,所有需要CGO_ENABLED=0禁用功能

# 例如windows编译linux可以运行的可执行程序修改如下环境变量
$Env:CGO_ENABLED=0;$Env:GOARCH="amd64";$Env:GOOS="linux"
go build [-o 二进制文件名称] [go源码文件]

# 例如linux编译windows可以运行的可执行程序修改如下环境变量
CGO_ENABLED=0 GOARCH=amd64 GOOS=windows go build [-o 二进制文件名称] [go源码文件]

~~~