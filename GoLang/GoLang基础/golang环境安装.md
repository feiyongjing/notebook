# 官网下载安装包：https://golang.google.cn/doc/install

### 安装参考：https://blog.csdn.net/qq_44830881/article/details/123457805
~~~shell
# 安装完成后检测环境变量
# PATH    go命令的路径，即 Go 开发包的安装目录下的bin目录

# 查看其他配置
go env

# GOPATH  Go语言工作目录，指定存放自已编写的go项目，包，编译的二进制文件等
# GOROOT  指定Go 开发包的安装目录
# GOPROXY 配置go下载包的代理地址为七牛云的go代理地址。go依赖包默认下载地址是国外的，中国访问不了
~~~