### 闭包使用
~~~go
package main

import "fmt"

// 闭包的本质是函数与局部变量的结合，感觉上类似于一个对象
// 闭包会返回一个函数，并且这个函数会使用这个闭包内部的局部变量，类似于一个对象，并且会暴露一个函数，函数内部会使用对象的若干成员变量，而一旦成员变量发生变化，下一次调用函数时使用的对象成员变量的值也会发生变化
// 闭包实现累加器
func F1() func(int) int {
	var n int = 0
	return func(a int) int {
		n += a
		return n
	}
}

func main() {
	// 初始化闭包，返回闭包内部的函数
	fn := F1()

	// 调用闭包函数进行累加
	fmt.Printf("累加器加1，结果是%v\n", fn(1))
	fmt.Printf("累加器继续加2，结果是%v\n", fn(2))
	fmt.Printf("累加器继续加3，结果是%v\n", fn(3))
	fmt.Printf("累加器继续加4，结果是%v\n", fn(4))
}

~~~