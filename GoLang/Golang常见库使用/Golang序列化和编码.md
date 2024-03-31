### 序列化、反序列化、base64编码和解码
~~~go
package main

import (
	"encoding/base64"
	"encoding/json"
	"fmt"
)

// 设置结构体json序列化标签，即json序列化使用这个标签作为属性的name
// 这是由于有些情况下json序列号后的字符串需要的不是首字母大写的字段名称或者不是这个名称而是其他的名称
type User struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
	Age  int32
	T1   `json:"t1"` //表示用t1字段包一层结构体
}

type T1 struct {
	FieldInt     int    `json:"field_int"`
	FieldIgnore  int    `json:"-"`                       //忽略指定进行序列化
	FieldBooleab bool   `json:"field_boolean,string"`    //序列化为其他不同的类型
	FieldString1 string `json:"field_string1,omitempty"` //空值时该字段不进行序列化，如果是复合类型想要为空时不序列化则字段必须是复合的指针类型
	FieldString2 string `json:"field_string2,omitempty"`
}

func main() {
	user := User{Id: 1, Name: "张三", Age: 18, T1: T1{FieldInt: 2, FieldIgnore: 19, FieldBooleab: true, FieldString2: "哈哈"}}

	// json序列化：对象转字符串，注意获取的是字节数组
	bytes, _ := json.Marshal(user)
	fmt.Printf("序列化的字符串是：%s \n", string(bytes))

	// json反序列化：字符串转对象，注意是字节数组转对象
	u1 := User{}
	json.Unmarshal(bytes, &u1)
	fmt.Printf("反序列化的对象是：%+v\n", u1)

	// base64编码
	str1 := base64.StdEncoding.EncodeToString(bytes)
	fmt.Printf("base64对字节数组编码结果是：%s\n", str1)

	// base64解码
	bytes1, _ := base64.StdEncoding.DecodeString(str1)
	fmt.Printf("对字符串进行base64解码结果是：%s \n", bytes1)
}
~~~
