### 自定义error结构体使用
~~~go
package main

import (
	"fmt"
)

type cusError struct {
	Code    int64
	Message string
}

// error的结构体必须实现 Error() 函数，否则不是 error
func (err cusError) Error() string {
	return fmt.Sprintf("错误码是：%d，错误信息是：%s\n", err.Code, err.Message)
}

func main() {

	r, err := Sum(-1, -4)
	// 判断是否出现错误
	if err != nil {
		// 类型断言判断错误类型是否是自定义的错误
		_, ok := err.(cusError)
		if ok {
			fmt.Println(err)
		}
	}
	fmt.Println(r)

}

func Sum(a int, b int) (r int, err error) {
	if a < 0 && b < 0 {
		// 默认的内置错误抛出方式1： errors.New("错误消息")
		// 默认的内置错误抛出方式2： panic("错误消息")
		// 抛出自定义的错误
		err = cusError{
			Code:    2233,
			Message: "求和结果不能是负数",
		}
		return
	}
	r = a + b
	return
}
~~~

# go的异常捕获

## defer 关键字
defer 关键字声明一个延迟调用函数，该函数可以是有名称的函数也可以是匿名函数
defer 延迟函数调用的时间是正常函数调用return 之后但是还未返回给上层函数之前
defer 延迟函数可以改变函数return的返回值
defer 延迟函数无论函数是否正常返回还是异常结束都会调用，可以使其资源释放
defer 可以对于一个正常函数指定多个延迟函数，它们的执行顺序是后进先出，即后声明的延迟函数先执行
defer 与 recover 配合使用实现异常的捕获和处理
不建议在for循环中使用defer

## recover 关键字
recover 是GO的内建函数，可以让宕机流程中的 goroutine 恢复过来
recover 只在defer延迟函数中有效，在其他函数中使用会返回nil，并且没有其他任何效果
如果当前的 goroutine 出现 panic ，调用recover 可以捕获到panic 的输入值，并且恢复正常的执行


## panic 解释
panic 是Go的一种异常机制，可以通过 panic 函数主动抛出异常


### 延迟函数执行的顺序：后进先出
~~~go
package main

import "fmt"

func main() {
	F1()
}

func F1() {
	fmt.Println("开始执行函数F1")

	defer func() {
		fmt.Println("后调用的匿名延迟函数B")
	}()

	defer F2()

	defer func() {
		fmt.Println("先调用的匿名延迟函数A")
	}()

	fmt.Println("结束执行函数F1")
}

func F2() {
	fmt.Println("执行具名函数F2")
}
~~~

### 延迟函数的参数传递问题
~~~go
package main

import "fmt"

func main() {
	F1()
}

func F1() {
	i := 1

	defer func(a int) {
		fmt.Printf("传递的参数是延迟函数在被加载时就进行复制的值：%d\n", a)
	}(i + 1)

	defer func() {
		fmt.Printf("不传递的参数是延迟函数在函数返回之前的值：%d\n", i)
	}()

	i = 99
	fmt.Printf("函数的返回值是：%d\n", i)
}
~~~

### 延迟函数的返回值问题
~~~go
package main

import "fmt"

func main() {
	i, j := F1()

	// 返回值的拷贝是无法修改的，但是全局变量和指针指向的值是可以修改的
	fmt.Printf("i：%d，j：%d，a：%d\n", i, *j, a)
}

var a = 100

func F1() (int, *int) {

	defer func() {
		a = 200
	}()

	return a, &a
}
~~~

### 延迟函数抓取异常，并且处理异常或者忽略异常
~~~go
package main

import "fmt"

func main() {
	F1()
}

func F1() {

	defer func() {
		// recover捕获异常恢复协程
		err := recover()
		if err != nil {
			fmt.Printf("捕获到了F1函数执行出现的异常，异常信息是：%v\n", err)
			fmt.Printf("处理F1函数执行出现的异常\n")
		} else {
			fmt.Println("忽略F1函数执行出现的异常")
		}
	}()

	fmt.Println("开始执行F1函数")

	panic("F1函数执行出现异常，异常原因xxx")

	fmt.Println("结束执行F1函数")
}
~~~

### 保证关闭资源
~~~go
package main

import (
	"fmt"
	"io"
	"log"
	"os"
)

func main() {
	F1()
}

func F1() {

	file, err := os.Open("go.mod")

	if err != nil {
		log.Fatal("文件打开失败，打印问题并退出程序")
	}

	// 延迟调用文件关闭
	defer file.Close()

	buf := make([]byte, 1024)
	for true {
		// 每次读取文件内容到buf中，返回读取的字符数量
		n, err := file.Read(buf)
		if err != nil && err != io.EOF {
			log.Fatal("文件打开失败，打印问题并退出程序")
		}
		if n == 0 {
			break
		}
		fmt.Println(string(buf[:n]))
	}
}
~~~
