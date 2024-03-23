## 泛型的限制
- 匿名结构体与匿名函数不支持泛型
- 不支持类型断言
- 不支持泛型方法，只能通过receiver来实现方法的泛型处理
- ~后的类型必须为基本类型，不能为接口类型

## 泛型使用例子
~~~go
package main

import (
	"fmt"
)

// 泛型结构体
type User[N interface{ string }, A interface{ int }] struct {
	Name N
	Age  A
}

// 对象泛型方法
func (u *User[N, A]) GetName() N {
	return u.Name
}

func (u *User[N, A]) SetName(name N) {
	u.Name = name
}

func (u *User[N, A]) SetAge(age A) {
	u.Age = age
}

func main() {
	// 泛型函数使用
	fmt.Println(F1(3, 7))

	var a, b int32 = 8, 19
	var a1, b1 MyInt32 = 9, 19
	var a2, b2 MyInt64 = 10, 19
	fmt.Println(F2(a, b))
	fmt.Println(F2(a1, b1))
	fmt.Println(F2(a2, b2))

	// 泛型结构体使用
	user := User[string, int]{Name: "张三", Age: 18}
	fmt.Println(user)
	user.SetName("李四")
	fmt.Println(user)

	// 泛型接口使用
	fmt.Println(F3(&user, 12))
}

// 泛型函数，该泛型支持 int 和 float32
func F1[T interface{ int | float32 }](a, b T) (c T) {
	if a >= b {
		c = a
	} else {
		c = b
	}
	return
}

// 泛型函数，自定义泛型支持的接口类型
func F2[T interface{ CusNumT }](a, b T) (c T) {
	if a >= b {
		c = a
	} else {
		c = b
	}
	return
}

// 泛型函数，自定义泛型支持的接口类型且定义接口函数
func F3[T interface{ CusNumT1[string, int] }](u T, a int) (c T) {
	u.SetAge(a)
	return u
}

// 自定义类型 CusNumT
type CusNumT interface {
	// 定义该接口支持的类型，~int64 表示支持int64的衍生类型，例如下面的 MyInt64
	// | 表示并集 换行表示取交集
	uint8 | int32 | float64 | ~int64
	uint16 | int32 | float64 | ~int64
}

// 自定义类型 MyInt64，该类型是int64的衍生类型，是具有基础类型 int64 的新类型，和int64 是不同的类型
type MyInt64 int64

// 自定义类型别名 MyInt32，该别名与int32是同一类型，所有的使用都和 int32 一样
type MyInt32 = int32

// 自定义泛型接口 CusNumT1
type CusNumT1[N string, A int] interface {
	any             // 该接口支持的类型
	SetAge(age int) // 该接口支持的类型需要有该函数
}
~~~