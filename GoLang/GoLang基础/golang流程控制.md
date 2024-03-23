## 条件判断和循环语句
~~~go
package main

import (
	"fmt"
)

func main() {

	fmt.Println(F1(0))
	fmt.Println(F2(6))
	fmt.Println(F3(100))
	fmt.Println(F4())

}

func F1(a int) (b int) {
	if a > 0 {
		b = a * 10
	} else if a == 0 {
		b = a + 10
	} else if a < 0 {
		b = a / 10
	}
	return
}

func F2(a int) (grade string) {
	var b int
	if a%3 > 0 {
		b = a/3 + 1
	} else {
		b = a / 3
	}

	switch b {
	case 1:
		grade = "春"   // 默认每个case的语句块中最后都会指向break，如果想要穿透到下一个case语句块中请使用fallthrough，注意穿透到下一个case并不会判断下一个的case条件而是直接指向代码块
	case 2:
		grade = "夏"
	case 3:
		grade = "秋"
	default:
		grade = "冬"
	}

	var x interface{} = "可惜不是你"

	switch x.(type) {
	case int:
		fmt.Printf("x 是 int 型\n") //  Type Switch 语句的 case 子句中不能使用fallthrough
	case float64:
		fmt.Printf("x 是 float64 型\n")
	case func(int) float64:
		fmt.Printf("x 是 func(int) 型\n")
	case bool, string:
		fmt.Printf("x 是 bool 或 string 型\n")
	default:
		fmt.Printf("未知型")
	}

	return
}

func F3(a int) (sum int) {
	for i := 0; i <= a; i++ {
		sum += i
	}
	return

	// 条件循环，即 for 后面是一个条件表达式
	//m, n := 1,10
	//for m!=n {
	//	m++;
	//}

	// 死循环
	// for{}
}

func F4() string {
	list := []int{2, 5, 1, 4, 0, 6, 9}
	for index, data := range list { // 遍历数组索引和数据
		fmt.Printf("inedx=%d, data=%d\n", index, data)
	}

	m := map[string]int{"a": 2, "b": 5, "c": 1, "d": 4, "e": 0, "f": 6, "g": 9}
	for key, value := range m { // 遍历map的KV对，如果k或者v不想要获取可以使用 _ 替代
		fmt.Printf("key=%d, value=%d\n", key, value)
	}

	str := "今天天气很差，难受吗？我不难受"
	for index, char := range str { // 遍历数组索引和数据
		fmt.Printf("inedx=%d, char=%c\n", index, char)
	}
	return "再切掉一点良心换一点野心"
}
~~~