# go连接redis使用
- 参考：https://redis.uptrace.dev/zh/guide/

### 安装golang 的redis包
~~~shell
# 在项目目录下执行
go get github.com/go-redis/redis/v8
~~~


### redis库的API使用
~~~go
package main

import (
	"context"
	"fmt"
	"github.com/go-redis/redis/v8"
)

func main() {

	// 连接方式一
	rdb := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // 没有密码，默认值
		DB:       10, // 默认DB 0
	})

	// 连接方式二
	//opt, err := redis.ParseURL("redis://<user>:<pass>@localhost:6379/<db>")
	//if err != nil {
	//	panic(err)
	//}
	//rdb := redis.NewClient(opt)

	defer rdb.Close()

	// go-redis 支持 Context，你可以使用它控制 超时 或者传递一些数据, 也可以 监控 go-redis 性能。
	ctx := context.Background()

	// 调用API执行对应的指令
	//val, _ := rdb.Get(ctx, "key").Result()
	//fmt.Println(val)

	// Do函数可以手动设置任意指令
	//val, err := rdb.Do(ctx, "get", "key").Result()
	//if err != nil {
	//	// redis.Nil 是一种特殊的错误，严格意义上来说它并不是错误，而是代表一种状态，
	//	// 例如你使用 Get 命令获取 key 的值，当 key 不存在时，返回 redis.Nil。在其他比如 BLPOP 、 ZSCORE 也有类似的响应
	//	if err == redis.Nil {
	//		fmt.Println("key does not exists")
	//		return
	//	}
	//	panic(err)
	//}
	// 手动断言获取的结果类型
	//fmt.Println(val.(string))

	val, err := rdb.Do(ctx, "hset", "testHashKey", "name", "张三").Result()
	fmt.Printf("存放hash数据进入redis的返回值是：%v，错误是：%v\n", val, err)

	// 通过API进行断言结果类型
	val = rdb.Do(ctx, "hget", "testHashKey", "name").String()
	fmt.Printf("存放hash数据进入redis的返回值是：%v\n", val)
}
~~~