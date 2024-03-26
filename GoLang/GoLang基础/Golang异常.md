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
		_, ok := err.(cusError)
		if ok {
			fmt.Println(err)
		}
	}
	fmt.Println(r)

}

func Sum(a int, b int) (r int, err error) {
	if a < 0 && b < 0 {
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
defer 可以对于一个正常函数指定多个延迟函数，它们的执行顺序是后进先出
defer 与 recover 配合使用实现异常的捕获和处理
不建议在for循环中使用defer

## recover 关键字
recover 是GO的内建函数，可以让宕机流程中的 goroutine 恢复过来
~~~go
~~~