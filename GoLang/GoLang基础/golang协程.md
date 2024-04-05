# goroutine 协程
- go中主进程可以启动多个线程，一个线程可以启动多个协程，go中一个线程可以轻松的启动上万个协程
- 物理线程作用在cpu上非常耗费资源，是重量级的。而协程是轻量级的线程，是逻辑态的
- go的协程特点：共享程序的堆空间、独立的栈空间、用户控制调度、协程是轻量级的线程

## 协程使用
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

## 协程异常处理
- 多个协程只要有一个出现panic或者error都会导致所有的协程包括主程序都退出程序
- 对于协程异常需要手动设置 defer延迟函数和recover函数进行捕获处理异常，不要让该异常错误影响到其他的协程和主程序
~~~go
package main

import (
	"fmt"
	"time"
)

// 每隔一秒打印一次
func sayHello(a int) {
	for i := 0; i <= a; i++ {
		fmt.Printf("sayHello函数第%v次打印hello\n", i)
		time.Sleep(time.Second)
	}
}

func test() {
	defer func() {
		err := recover()
		if err != nil {
			fmt.Printf("test() 函数发生了异常错误，错误是：%v\n", err)
		}
	}()

	// 这里的map没有使用make创建赋值，所有直接使用这个may会panic报错
	var myMap map[int]string
	myMap[0] = "哈哈"
}

func main() {
	go sayHello(5)
	go test()

	// 主线程每隔一秒打印一次，防止主线程运行太快导致协程还没有执行就退出
	for i := 0; i <= 5; i++ {
		fmt.Printf("main函数第%v次打印hello\n", i)
		time.Sleep(time.Second)
	}
	fmt.Println("主线程正常退出")
}
~~~

### 协程带来的问题
- 和线程一样，存在多个协程向共享变量写数据时存在数据错乱，解决方案是使用锁或者channel管道解决

### 互斥锁解决协程共享变量问题
- 缺点是主线程睡眠一段时间之后不确定协程是否执行完毕然后释放锁，并且主线程的睡眠时间不确定，必须多处加锁和解锁
~~~go
package main

import (
	"fmt"
	"sync"
	"time"
)

var myMap map[int]int = make(map[int]int)
var lock sync.Mutex // sync.Mutex 是互斥锁

// 计算阶乘，太大的数会越界变为负数甚至是0
func test(a int) {
	res := 1
	for i := 1; i <= a; i++ {
		res *= i
	}

	lock.Lock()
	myMap[a] = res
	lock.Unlock()
}

func main() {
	// 开200个协程计算阶乘
	for i := 0; i <= 200; i++ {
		go test(i)
	}

	// 睡眠10s
	time.Sleep(10 * time.Second)

	// 为啥写数据完成后，读数据时还是需要加锁？
	// 原因是主线程不确定在睡眠的10s内写数据操作一定会完成，如果没有完成则读取不到完整的数据，而添加锁可以阻塞等待所有协程释放锁，直到获得锁进行读数据
	lock.Lock()
	for i, v := range myMap {
		fmt.Printf("myMap[%v]=%v\n", i, v)
	}
	lock.Unlock()
}
~~~


