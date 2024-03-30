## golang函数
~~~go
package main

import (
	"errors"
	"fmt"
)

func main() {
	s, err := sum(100, 899)
	fmt.Println(s, err)
}

// 函数名称后第一对括号中的是函数参数列表，第二对括号中是函数的返回值列表（golang支持多个返回值，如果只有一个返回值则括号可以省略，默认约定返回值在有error类型就优先检查error返回值确定函数是否调用出现错误）
// 注意指针类型的返回值，如果有名称的情况下该指针默认是指向的nil，即null
func sum(a int, b int) (sum int, err error) {
	if a < 0 && b < 0 {
		err = errors.New("两数相加不能小于0")
		return 0, err // 如果没有给定返回值的名称，而是只有返回值列表的类型，可以使用return 按照顺序传递返回值列表返回
	}
	sum = a + b // 如果有给定返回值的名称，可以直接给返回值名称赋值，后续只需要return，不用写返回值列表
	return
}
~~~

## 面向对象函数
~~~go
package main

import (
	"fmt"
)

type User struct {
	Name string
	Age  int
}

func main() {
	user := NewUser("张三", 18)
	fmt.Println(user)

	user.SetName("李四")
	fmt.Println(user)
}

func NewUser(name string, age int) *User {
	// 创建对象方式一
	return &User{Name: name, Age: age}

	// 创建对象方式二
	//user := new(User)
	//user.Name = name
	//user.Age = age
	//return user
}

// 函数名前面的括号是设置设置那些类型的对象可以调用该函数
// 注意如果是修改修改调用者对象本身的成员请一定要指定为该调用者类型指针才能修改
func (u *User) GetName() string {
	return u.Name
}

func (u *User) SetName(name string) {
	u.Name = name
}
~~~