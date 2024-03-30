## 常见类型
### 基本数据类型，没有赋值都是有默认的值
- bool 布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true
- int、uint、int8、uint8、int16、uint16、int32、uint32、int64、uint64 例如 var i int = 100
- float32、float64、complex、complex64、complex128
- byte 字符型
- string 例子：var b string = "true"

### 派生类型，没有值默认是nil，即null
- struct结构体类型
- 固定长度数组 例子 var arr [5]int= [5]int{1, 2, 3, 4, 5} 或 arr1 := [...]int{1, 2, 3, 4, 5}
- slice切片（可变长度数组） 例子 s := new ([]int) 注意返回的是切片的指针
- map集合类型 例子 m := make(map[KeyType]ValueType, 10)
- pointer指针类型 例子 var a *int
- interface接口类型，interface 类型默认是指针类型，interface{}是空接口没有任何函数，所有的类型都实现了空接口
- 函数类型
- chan管道类型 例子 ch := make(chan int, 10)

### 值类型和引用类型
- 值类型包含基本数据类型、struct结构体、数组，值类型一般都是使用new内置函数进行创建的。注意值类型都有对应的指针类型，而值类型的数据在被当成参数传递函数时是进行了值的拷贝，在函数内部修改是无法影响到函数外部的值的
- 引用类型包含 slice切片、map集合、pointer指针、interface接口、函数、chan管道，引用类型一般都是使用make内置函数进行创建的

## 常量
~~~go
package main

import (
	"fmt"
)

// 首字母大写的全局变量只要被其他地方引用了所在的包，那么就可以直接使用改全局变量
const (
	A = 100    // 对于直接赋值的常量来说是有自动类型推断的
	B = "哈哈" 
)

// 首字母小写的全局变量只在当前包中有效
const (
	a uint8= iota * 10   // iota 关键字用于常量中表示该常量是默认类型的默认值，后续的常量会使用当前常量的表达式但是iota表示的数会加1，如果有些常量不需要请使用 _ 表示不需要变量占位
	b
    _
    d
    e  
)

// 函数或代码块中的局部变量只在该作用域中有效
func main() {
	fmt.Println(A, B)
    fmt.Println(a, b, d, e)
}
~~~

## struct结构体
~~~go
package main

import "fmt"

// 结构体声明
type user struct {
	name string
	age  int
	int64  // 这是匿名字段，只设置字段类型，注意匿名字段的类型不能与其他字段的类型一致
}

func main() {
    // 结构体对象创建方式1
	u := user{name: "哈哈哈", age: 18}
	fmt.Println(u.name,u.age)
	
    // 结构体对象创建方式2：内置的new函数会申请一个指定类型大小的内存空间，返回所内存空间的指针
	u1 := new(user)
	u1.name="嘻嘻嘻"
	u1.age=20
	u1.int64=80  // 直接通过匿名字段的类型可以设置匿名字段的值
	fmt.Println(u1.name, u1.age, u1.int64)

}
~~~

## pointer指针
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

## slice切片（可变长度数组）
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

## 函数类型
~~~go
package main

import "fmt"

func Sum(a int, b int) int {
	return a + b
}

func F1(myFunc func(int, int) int, a int, b int) {
	fmt.Printf("传递函数对象和参数进行函数回调，回调结果是：%d\n", myFunc(a, b))
}

func main() {
	// 函数名就是代表该函数对象
	F1(Sum, 1, 2)

	// 匿名函数直接声明并调用
	func(a int, b int) { fmt.Printf("匿名函数直接声明并调用：%d\n", a*b) }(3, 7)
}
~~~

## channel 管道类型
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

## 类型转换，go中没有隐式的类型转换，只能手动进行类型断言判断，从而转换类型
~~~go
package main

import (
	"fmt"
	"strconv"
)

func main() {

	// 如果是一些确定的类型，可以通过对应的函数进行转换
	var a int8 = 13
	var b int32 = int32(a)
	var c byte = byte(b)
	var d string = fmt.Sprintf("%v", c)
	e, _ := strconv.Atoi(d)
	f, _ := strconv.ParseInt(d, 10, 64)

	// 如果是不确定的类型请进行类型断言转换类型
	var i interface{} = "hello"
	s, ok := i.(string)
	if ok {
		fmt.Println("string类型断言成功，并且转换赋值给了新的变量：%v\n", s)
	} else {
		fmt.Println("string类型断言失败")
	}

	fmt.Printf("a=%v,b=%v,c=%v,d=%q,e=%v,f=%v\n", a, b, c, d, e, f)
}
~~~


