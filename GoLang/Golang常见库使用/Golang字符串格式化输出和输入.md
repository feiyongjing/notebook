### 字符串格式化输出和输入
~~~go
package main

import (
	"fmt"
	"os"
)

type User struct {
	Id   int64
	Name string
	Age  int32
}

func main() {

	user := User{Id: 1, Name: "张三", Age: 18}

	fmt.Println("Println打印到标准输出并且自动换行")
	fmt.Sprintf("%s\n", "Sprintf格式化字符串并返回")
	fmt.Fprintf(os.Stdin, "%s\n", "Fprintf格式化字符串并写入指定的IO中")
	fmt.Printf("%s\n", "Printf打印格式化字符串到标准输出但是不自动换行，需要手动添加换行字符")
	fmt.Printf("go中值的类型默认显示：%v\n", user)
	fmt.Printf("go中值的类型默认显示，且包含字段名：%+v\n", user)
	fmt.Printf("go中对象的类型：%T\n", user)
	fmt.Printf("go中对象的类型、包含字段名的值：%#v\n", user)

	fmt.Printf("%q\n", "显示的字符串包裹双引号")
	fmt.Printf("打印百分号%%\n")

	// 整数和字符设置
	v1, v2 := 10, 20170
	fmt.Printf("数字二进制显示：%b\n", v1)
	fmt.Printf("按照 Unicode 码表显示字符：%c\n", v2)
	fmt.Printf("数字十进制显示：%d\n", v1)
	fmt.Printf("数字八进制显示：%o\n", v1)
	fmt.Printf("数字以0o前缀的八进制显示：%O\n", v1)
	fmt.Printf("按照 Unicode 码表显示字符，并且使用单引号包裹字符显示：%q\n", v2)
	fmt.Printf("数字十六进制小写显示：%x\n", v1)
	fmt.Printf("数字十六进制大写显示：%X\n", v1)
	fmt.Printf("显示对应的Unicode 格式：%U\n", v2)

	// 宽度设置
	fmt.Printf("%v 的二进制：%b; go语法表示二进制为：%#b;指定二进制宽度为8，不足8位补0：%08b \n", v1, v1, v1, v1)
	fmt.Printf("%v 的十六进制：%x; go语法表示十六进制为：%#x;指定十六进制宽度为8，不足8位补0：%08x \n", v1, v1, v1, v1)
	fmt.Printf("%v 的字符：%c; 指定二进制宽度为5，不足8位补0：%05c \n", v2, v2, v2)

	// 浮点型设置
	v3 := 3.1415926
	fmt.Printf("浮点型默认显示：%f\n", v3)
	fmt.Printf("科学计算法显示：%e\n", v3)

	// 指针地址
	str := "策马浪迹天涯 唤来风雨相伴自潇洒"
	bytes := []byte(str)
	fmt.Printf("指针的地址是：%p\n", bytes)

	// 读取标准输入
	var s1, s2 string
	fmt.Println("读取输入的一行字符并且打印，请输入你的字符串：")
	fmt.Scanln(&s1)
	fmt.Printf("你输入的字符串是：%s\n", s1)

	fmt.Println("读取输入的一行字符格式化字符串提前内容并且打印，例如输入name:xxx，会抓取xxx打印，请输入你的字符串：")
	fmt.Scanf("name:%s", &s2)
	fmt.Printf("从你输入的字符串提前name是：%s\n", s2)
}
~~~