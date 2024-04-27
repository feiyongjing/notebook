### Cobra简介
Cobra 提供了一种简单而强大的方式来构建命令行客户端，它能够帮助开发者快速实现复杂的命令行工具，并且提供了丰富的功能和选项，包括自动生成帮助文档、自动补全等


## 安装cobra依赖库和命令行插件
~~~shell
# 安装cobra依赖库
go get -u github.com/spf13/cobra@latest

# 安装cobra cli插件
go install github.com/spf13/cobra-cli@latest
~~~

## 创建项目
~~~shell
# 创建项目目录并进入该目录
mkdir -p myApp && cd myApp

# 初始化项目go.mod
go go mod init [项目名称]

# 项目目录下初始化项目
cobra-cli init [你的项目路径，不写默认是当前目录]
~~~

# 项目文件介绍

### main.go会去执行cmd/root.go的Execute函数，使用 go build main.go编译后就可以直接执行打包好的 main，如果需要也可以将打包文件改为自定义的命令名称
### cmd/root.go文件是根命令的详细执行内容，代码中有一个rootCmd变量
- Use: 自定义的命令名称
- Short：该命令的简短描述
- Long：该命令的详细描述,在帮助命令，如[appName] -h或者[appName] [command] -h时会显示此程序和此命令的字符串
- Run: 该命令执行入口，代码主要写在这个模块中
- Args: 对该命令的参数数量限制
    - cobra.MinimumNArgs(int)   函数当参数数目低于配置的最小参数个数时报错 
    - cobra.MaximumNArgs(int)   函数当参数数目大于配置的最大参数个数时报错 
    - cobra.RangeArgs(min, max) 函数当参数的个数不在区间 min 和 max 之中时报错 
    - cobra.ExactArgs(int)      函数如果参数数目不是配置的参数个数时报错
    - cobra.NoArgs              函数没有参数则报错 

## 创建新的子命令
~~~shell
# 运行如下命令会在cmd目录下创建一个和root.go类似模板的文件，文件名称是自定义的命令名称，里面记录了子命令的详细信息和执行方式
cobra-cli add [自定义的子命令]
# 例如root.go中的父命令是 myApp，而新的子命令是config，则该子命令的执行应该是 myApp config

# 在已有的自定义子命令之后继续添加子命令
cobra-cli add [自定义的子命令] -p [子命令后面继续加子命令，注意后缀必须是Cmd，最终的子命令是不会包含Cmd的]
~~~

## 为command(命令)添加选项(flags)，也就是常见key-value的命令行参数
### flags用来控制命令的具体行为分为本地(local) flags和全局(persistent) flags两类
- 对于全局类型的选项，既可以设置给该命令，又可以设置给该命令的子命令
- local 类型的选项只能设置给指定的 Command


## 全局flags参数设置
### 方式一：使用PersistentFlags().String添加flags
在root.go定义一个全局变量var foo *string，然后在root.go的init里面添加：
~~~go
foo =rootCmd.PersistentFlags().String("foo","defaultXXX","foo args help")
~~~
假设现在有serve 这个子命令
执行go run main.go serve --foo xxx，xxx被赋值给foo；如果go run main.go serve，默认的defaultXXX被赋值给foo,由此可见在父命令rootCmd定义的--foo全局flags在子命令serve里面也可以用，同理在子命令config和create里也可以使用这个自定义的--foo参数

### 方式二：使用PersistentFlags().StringVar添加flags
在root.go定义一个全局变量var print string，然后在root.go的init里面添加：
~~~go
rootCmd.PersistentFlags().StringVar(&print,"print","defaultPrint","print args help")
~~~
假设现在有serve 这个子命令
执行go run main.go serve --print ppp，ppp被赋值给print；如果go run main.go serve，默认的defaultPrint赋值给print,由此可见在父命令rootCmd定义的--print全局flags在子命令serve里面也可以用，同理在子命令config和create里也可以使用这个自定义的--print参数

### 方式三：PersistentFlags().BoolVarP添加flags
在root.go定义一个全局变量var show string，然后在root.go的init里面添加：
~~~go
// 下面定义了一个Flag show,show默认为false, 有两种调用方式--show\-w，命令后面接了show则上面定义的show变量就会变成true
rootCmd.PersistentFlags().BoolVarP(&show,"show","w",false,"show or w args help")
~~~
假设现在有serve 这个子命令
执行go run main.go serve -w，show为true；如果go run main.go serve，默认的show为false,由此可见在父命令rootCmd定义的--w/--show全局flags在子命令serve里面也可以用，同理在子命令config和create里也可以使用这个自定义的--show\-w参数

## 本地local flags 参数设置
假设现在有serve 这个子命令，在cmd/serve.go的init函数中添加下面代码设置本地flags，作用和上述全局flags参数设置类似，但是只能用于serve这个子命令
~~~go
foo2 =serveCmd.Flags().String("foo2","fo2","foo2 args help")
// 需要提前在cmd/serve.go定义一个全局变量var print2 string
serveCmd.PersistentFlags().StringVar(&print2,"print2","defaultPrint2","print2 args help")
// 需要提前在cmd/serve.go定义一个全局变量var s string
serveCmd.Flags().BoolP("s1","s",false,"s1 or s args help")
~~~


# 例子一：添加一个子命令用于显示系统时间
使用 cobra-cli add showtime 创建并且修改后的cmd/Showtime文件如下
执行 go build -o myapp.exe .\main.go 打包出来myapp.exe文件，注意myapp就是cmd/root.go文件所设置的根命令
执行 .\myapp.exe showtime 就可以显示当前的系统时间了
go run .\main.go showtime 
~~~go
package cmd

import (
	"fmt"
	"time"
	"github.com/spf13/cobra"
)

// showtimeCmd represents the showtime command
var showtimeCmd = &cobra.Command{
	Use:   "showTime",
	Short: "显示当前的系统时间",
	Long: `显示当前的系统时间`,
	Run: Showtime,
}

// Showtime 显示当前时间
func Showtime(cmd *cobra.Command, args []string) {
	fmt.Println(time.Now())
}


func init() {
	rootCmd.AddCommand(showtimeCmd)
}
~~~


# 例子二：添加一个子命令用于显示系统时间按照指定字符串格式化后的时间
使用 cobra-cli add parse -p showtimeCmd 创建并且修改后的cmd/Parse文件如下
执行 go build -o myapp.exe .\main.go 打包出来myapp.exe文件，注意myapp就是cmd/root.go文件所设置的根命令
执行 .\myapp.exe showtime parse --format="2006-01-02 15:04:05" 就可以显示当前的系统时间格式化后的字符串了
或者直接执行 go run .\main.go showtime parse --format="2006-01-02 15:04:05" 运行
~~~go
package cmd

import (
	"fmt"
	"time"
	"github.com/spf13/cobra"
)

var format string

// parseCmd represents the parse command
var parseCmd = &cobra.Command{
	Use:   "parse",
	Short: "显示系统时间按照指定字符串格式化后的时间",
	Long:  "显示系统时间按照指定字符串格式化后的时间",
	Run: func(cmd *cobra.Command, args []string) {
		// 注意golong不是使用 yyyy-MM-dd HH:mm:ss 来表示格式化时间字符串，而是2006-01-02 15:04:05 这个时间字符串来格式化时间
		// 原因是这是golang的诞生时间
		fmt.Println(time.Now().Format(format))
	},
}

func init() {
	showtimeCmd.AddCommand(parseCmd)

	parseCmd.Flags().StringVarP(&format, "format", "f", "", "format 参数是时间格式，例如\"2006:01:02 15:04:05\"")
	// 这里指定format flag为必须
	_ = parseCmd.MarkFlagRequired("format")
}
~~~




