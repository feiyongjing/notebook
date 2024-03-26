### 简单使用
~~~go
package main

import (
	"fmt"
	"regexp"
)

func main() {

	mustCompile := regexp.MustCompile(`[a-z]+[0-9]+`)
	fmt.Println(mustCompile.MatchString("abc123"))

	findAll := mustCompile.FindAll([]byte("a3xk2s3a322axas2"), -1)
	fmt.Println(string(findAll[0]))
}
~~~