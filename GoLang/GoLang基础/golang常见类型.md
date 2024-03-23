### 常见类型
- bool 布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true
- int、uint、int8、uint8、int16、uint16、int32、uint32、int64、uint64 例如 var i int = 100
- float32、float64、complex、complex64、complex128
- string 例子：var b string = "true"
- 固定长度数组 例子 var arr [5]int= [5]int{1, 2, 3, 4, 5} 或 arr1 := [...]int{1, 2, 3, 4, 5}
- 切片（可变长度数组） 例子 s := new ([]int) 注意返回的是切片的指针
- map集合类型 例子 m := make(map[KeyType]ValueType, 10)
- 指针类型 例子 var a *int
- 接口类型 例子 var a interface{}
- chan管道类型 例子 ch := make(chan int, 10)


### golang中基本数据类型变量：包含数组、结构体，没有赋值都是有默认的值。而引用类型：接口、指针、切片（可变长度数组）没有值默认是nil，即null

## 常量
~~~go
package main

import (
	"fmt"
)

const (
	A = 100    // 对于直接赋值的常量来说是有自动类型推断的
	B = "哈哈" 
)

const (
	a uint8= iota * 10   // iota 关键字用于常量中表示该常量是默认类型的默认值，后续的常量会使用当前常量的表达式但是iota表示的数会加1，如果有些常量不需要请使用 _ 表示不需要变量占位
	b
    _
    d
    e  
)

func main() {
	fmt.Println(A, B)
    fmt.Println(a, b, d, e)
}
~~~

## 结构体
~~~go
package main

import "fmt"

// 结构体声明
type user struct {
	name string
	age  int
}

func main() {
    // 结构体对象创建方式1
	u := user{name: "哈哈哈", age: 18}
	fmt.Println(u.name,u.age)
	
    // 结构体对象创建方式2
	u1 := new(user)
	u1.name="嘻嘻嘻"
	u1.age=20
	fmt.Println(u1.name, u1.age)
}
~~~

## 指针
~~~go
package main

import (
	"fmt"
)

// 程序入口
func main() {
	var a int = 100
	f1(&a)   // 函数传递a的指针，函数内部通过指针修改变量a
	fmt.Println(a)
}

func f1(a *int) {
	*a = *a - 1
}
~~~

## 切片（可变长度数组）
~~~go
package main

import "fmt"

func main() {

	// 创建切片方式1
	var s []int
	s = append(s, 99)

	// 创建切片方式2
	s1 := new([]int)  // 注意返回的是数组的指针
	*s1 = append(*s1, 199)  // 切片追加元素，可以追加多个元素，超出容量是会自动扩容的

	// 创建切片方式3，定义切片的初始长度和容量
	s2 := make([]int, 10, 20)
	s2 = append(s2, 299，123，122，111，388)
    fmt.Printf("s2切片的长度=%d s2切片的容量=%d\n",len(s2),cap(s2))

	fmt.Println(s, s1, s2)

    // 切片的截取，和python的list截取类似
    fmt.Println(s2[10:])
}
~~~

## map集合
~~~go
package main

import "fmt"

func main() {
	// 创建map，默认go会选择合适的初始空间
	m := make(map[string]int)

	// 创建map，指定初始空间为10
	// m := make(map[string]int, 10)

	// 创建map并初始化数据
	//m := map[string]int{
	//	"apple": 1,
	//	"banana": 2,
	//	"orange": 3,
	//}

	// 增加和修改键值对
	m["apple"] = 5
	m["banana"] = 8
	m["orange"] = 10
	fmt.Println(m)

	// 获取键值对
	v1 := m["apple"]
	v2, ok := m["pear"] // 如果键不存在，ok 的值为 false，v2 的值为该类型的默认值
	fmt.Println(v1, v2, ok)

	// 遍历 Map
	for k, v := range m {
		fmt.Printf("key=%s, value=%d\n", k, v)
	}

	// 获取 Map 的长度
	l := len(m)
	fmt.Println(l)

	// 删除键值对
	delete(m, "banana")
	fmt.Println(m)
}
~~~

## chan 管道类型
~~~go
package main

import "fmt"

func main() {

	// 创建读写管道，管道内部传输的数据类型是int，缓冲区大小是10个字节
	ch := make(chan int, 10)

	// 创建只写管道，管道内部传输的数据类型是int，缓冲区大小是10个字节
	ch1 := make(chan <-int, 10)

	// 创建只读管道，管道内部传输的数据类型是int，缓冲区大小是10个字节
	ch2 := make(chan int, 10)

	fmt.Println(ch, ch1, ch2)
}
~~~



