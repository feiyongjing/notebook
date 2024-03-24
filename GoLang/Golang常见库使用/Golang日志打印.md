## 基础日志打印
~~~go
package main

import (
	"log"
	"os"
)

// 初始化设置
func init() {
	log.SetFlags(log.LstdFlags | log.Llongfile) // 设置打印出的日志显示日期时间和打印日志的文件完整路径和行号
	log.SetOutput(os.Stderr)                    // 设置日志打印到哪里，这里设置的标准错误
}

func main() {

	log.Println("打印日志并且自动换行")
	log.Printf("%s\n", "打印格式化字符串日志")
	log.Fatalf("%s\n", "Fatalf函数打印格式化字符串日志并且会导致退出程序")

}
~~~