# goroutine 协程
- go中主进程可以启动多个线程，一个线程可以启动多个协程，go中一个线程可以轻松的启动上万个协程
- 物理线程作用在cpu上非常耗费资源，是重量级的。而协程是轻量级的线程，是逻辑态的
- go的协程特点：共享程序的堆空间、独立的栈空间、用户控制调度、协程是轻量级的线程

# 协程使用
~~~go
package main

import (
	"fmt"
	"time"
)

// 每隔一秒打印一次
func test(a int) {
	for i := 0; i <= a; i++ {
		fmt.Printf("test函数第%v次打印hello\n", i)
		time.Sleep(time.Second)
	}
}

func main() {

	a := 10

	// go 关键字直接开启协程直接函数代码
	go test(a)

	// 主线程每隔一秒打印一次，防止主线程运行太快导致协程还没有执行就退出
	for i := 0; i <= a; i++ {
		fmt.Printf("main函数第%v次打印hello\n", i)
		time.Sleep(time.Second)
	}
}
~~~