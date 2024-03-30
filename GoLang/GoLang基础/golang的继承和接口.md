# go中interface是隐式的实现
- go中的结构体对象实现interface 无需任何关键字，只需要该对象的方法集中包含接口定义的所有方法且方法签名一致
- 对象实现接口可以借助struct 内嵌的特性，实现接口的默认实现，即组合使用结构体
- 类型T方法集包含全部receiver T方法，类型 *T 方法集包含 receiver T + *T 方法
- 类型T对象的 value或pointer（指针）可以调用全部的方法，编译器会自动转换
- 类型T实现接口，不管T还是*T都实现了该接口
- 类型*T实现接口，只有T类型的指针实现了该接口

### 继承使用
~~~go
package main

import "fmt"

// 定义父类型
type animal struct {
	Name string
	Age  int32
}

func (p animal) GetName() string {
	return ""
}

func (p animal) SetAge(age int32) {}


type User struct {
	Id     int64
	animal // 直接继承父类型，本质是一个匿名的字段，该字段的类型是继承的父类型
}

func (u User) GetId() int64 {
	return u.Id
}

// 重写父类函数
func (u *User) GetName() string {
	return u.Name
}

func (u *User) SetAge(age int32) {
	u.Age = age
}

func main() {
	user := User{Id: 1, animal: animal{Name: "张三", Age: 10}}
	up := &user

	fmt.Printf("对象实例直接调用GetId方法，结果是：%d\n", user.GetId())
	fmt.Printf("对象指针直接调用GetId方法，结果是：%d\n", up.GetId())

	fmt.Printf("对象实例直接调用GetName方法，结果是：%s\n", user.GetName())
	fmt.Printf("对象指针直接调用GetName方法，结果是：%s\n", up.GetName())

	user.SetAge(20)
	fmt.Printf("对象实例直接调用SetAge方法，结果是：%v\n", user)
	up.SetAge(13)
	fmt.Printf("对象指针直接调用SetAge方法，结果是：%v\n", up)
}
~~~

### 多继承使用
~~~go
package main

import "fmt"

// 定义父类型
type Animal struct {
	Name string
}

func (a *Animal) Eat() {
	fmt.Printf("动物在吃饭\n")
}

type Person struct {
	Age int
}

func (a *Person) Eat() {
	fmt.Printf("人在吃饭\n")
}

// 多继承
type User struct {
	Id     int64
	Animal // 继承父类型1
	Person // 继承父类型2
}

func (u *User) Eat() {
	fmt.Printf("用户在吃饭\n")
}

// 重写父类函数
func (u *User) GetName() string {
	return u.Name
}

func main() {
	user := User{Id: 1, Animal: Animal{Name: "张三"}, Person: Person{Age: 18}}

	// 如果在多继承中出现了相同的函数，并且没有重写该函数的情况下需要明确指定调用哪个父类的函数
	// 如果在多继承中出现了相同的字段，并且子类没有定义相同的字段需要明确指定调用哪个父类的字段
	user.Animal.Eat()
	user.Person.Eat()

	// 如果已经子类已经定义与父类相同的字段或者重写父类函数则默认会调用子类的函数或者字段
	user.Eat()
}
~~~

### 接口使用
- 一个结构体必须实现接口中所有定义的方法才是这个结构体实现了接口
- 在golang中接口不能有字段，但是一个接口可以继承另外的接口，而实现这个接口时也必须该接口继承是所有接口
~~~go
package main

import "fmt"

// 动物接口模拟定义动物的行为，
type AnimalInterface interface {
	Eat() // 描述吃的行为
}

// 定义动物的默认实现
type Animal struct {
	Name string
}

func (a Animal) Eat() {
	fmt.Printf("%v 在吃饭\n", a.Name)
}

// 继承动物类重写函数就实现了接口
type Cat struct {
	Id     int64
	Animal // 直接继承父类型
	Age    int32
}

func (c *Cat) SetName(name string) {
	c.Name = name
}

// 重写接口函数
func (c *Cat) Eat() {
	fmt.Printf("姓名为%v 的猫在吃饭\n", c.Name)
}

func main() {
	cat := Cat{Id: 1, Animal: Animal{Name: "小白"}, Age: 10}
	cat.Eat()
	
	cat.SetName("小黑")
	cat.Eat()
}

~~~
