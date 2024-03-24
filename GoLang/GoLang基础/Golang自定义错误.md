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