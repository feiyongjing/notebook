### 序列化、反序列化、base64编码和解码
~~~go
package main

import (
	"encoding/base64"
	"encoding/json"
	"fmt"
)

type User struct {
	Id   int64
	Name string
	Age  int32
}

func main() {
	user := User{Id: 1, Name: "张三", Age: 18}

	// json序列化：对象转字符串，注意获取的是字节数组
	bytes, err := json.Marshal(user)
	fmt.Printf("序列化的字符串是：%s ，错误是：%+v\n", bytes, err)

	// json反序列化：字符串转对象，注意是字节数组转对象
	u1 := User{}
	json.Unmarshal(bytes, &u1)
	fmt.Printf("反序列化的对象是：%+v\n", u1)

	// base64编码
	str1 := base64.StdEncoding.EncodeToString(bytes)
	fmt.Printf("base64对字节数组编码结果是：%s\n", str1)

	// base64解码
	bytes1, err1 := base64.StdEncoding.DecodeString(str1)
	fmt.Printf("对字符串进行base64解码结果是：%s ，错误是：%+v\n", bytes1, err1)
}
~~~
