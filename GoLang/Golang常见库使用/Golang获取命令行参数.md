### go 获取启动时的命令行参数
~~~go
package main

import (
	"fmt"
	"os"
)

func main() {
	// os.Args 获取go 程序启动时传入的参数，默认数组第一个参数是 go 程序的可执行文件路径
	for i, arg := range os.Args {
		fmt.Printf("Arg[%v]=%v\n", i, arg)
	}
}
~~~

### 使用 flag 包进行解析go 程序启动时的命令行参数
- 命令行参数的输入先后顺序没有影响
~~~go
package main

import (
	"flag"
	"fmt"
)

func main() {
	// 定义命令行参数
	var host string
	var port int
	var user string
	var pwd string

	// 设置各个命令行参数的信息
	// 第一个参数是设置该命令行参数的值赋值给那个变量
	// 第二个参数是设置该命令行参数的key，例如 h 就是 -h 的意思
	// 第三个参数是设置该命令行参数的默认值
	// 第四个参数是设置命该令行参数的说明
	flag.StringVar(&host, "h", "127.0.0.1", "主机必须输入")
	flag.IntVar(&port, "port", 3306, "端口号必须输入")
	flag.StringVar(&user, "u", "", "用户名默认为空")
	flag.StringVar(&pwd, "pwd", "", "密码默认为空")

	// flag.Parse() 会获取 os.Args[1:] 的参数进行解析出flag设置需要的参数
	flag.Parse()

	// flag.Parsed() 是用于检查 flag.Parse() 是否已经调用过
	fmt.Printf("检查flag.Parse() 是否已经调用过：%v", flag.Parsed())

	fmt.Printf("输入的主机是：%v，输入的端口号是：%v，输入的用户名是：%v，输入的密码是:%v", host, port, user, pwd)
}
~~~




