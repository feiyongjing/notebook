### 反射基础使用
~~~go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int64
	Name string
	Age  int32
	Node *User
}

func main() {

	user := User{Id: 1, Name: "张三", Age: 18}
	user1 := User{Id: 2, Name: "李四", Age: 20, Node: &user}

	inspectStruct(user1)

	fmt.Printf("%+v\n", user1)
	// 修改结构体属性必须使用结构体指针（函数的传递会发生拷贝，不传递指针无法修改结构体）
	// Elem() 将指针Value转为非指针Value
	// Addr() 将非指针Value转为指针Value
	reflect.ValueOf(&user1).Elem().FieldByName("Name").SetString("gggg")
	fmt.Printf("%+v\n", user1)
	reflect.ValueOf(&user1).Elem().Addr().Elem().FieldByName("Name").SetString("xxxx")
	fmt.Printf("%+v\n", user1)
}

func inspectStruct(u interface{}) {

	t := reflect.TypeOf(u) // 获取反射值的类型
	fmt.Println(t)

	//获取结构体中指定的字段
	name, _ := t.FieldByName("Name")
	fmt.Printf("%+v\n", name)

	v := reflect.ValueOf(u)  // 获取反射值的对象
	numField := v.NumField() // 获取结构体对象中的属性数量

	for i := 0; i < numField; i++ {
		field := v.Field(i) // 获取结构体中第i个字段
		ft := field.Type()  // 获取非指针类型字段的类型
		ftName := ft.Name() // 获取非指针类型字段类型名称
		ftKind := ft.Kind() // 获取非指针类型字段的种类（kind）

		switch ftKind {
		case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
			fmt.Printf("field:%d type:%s value:%d\n", i, ftName, field.Int())

		case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64:
			fmt.Printf("field:%d type:%s value:%d\n", i, ftName, field.Uint())

		case reflect.Bool:
			fmt.Printf("field:%d type:%s value:%t\n", i, ftName, field.Bool())

		case reflect.String:
			fmt.Printf("field:%d type:%s value:%q\n", i, ftName, field.String())

		case reflect.Ptr:
			e := field.Elem()    // 获取指针类型字段的值
			ftPName := ft.Name() // 获取指针类型字段名称
			ftPKind := ft.Kind() // 获取指针类型字段的种类（kind）
			fmt.Printf("field:%d type:%s value:%+v kind:%s\n", i, ftPName, e, ftPKind)

		default:
			field.Interface() // 转换为interface{} 类型
			fmt.Printf("field:%d type:%s unhandled kind:%s value:%d\n", i, ftName, ftKind, field.Int())

		}
	}
}
~~~