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

## 互斥锁解决协程共享变量问题
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

## 协程同步：互斥锁实现生产者消费者有序生产消费，并且在主线程中通过sync.WaitGroup等待所有协程完成任务
~~~go
package main

import (
	"fmt"
	"sync"
)

func producer(intChan chan int, mutex sync.Mutex, count *int, waitGroup *sync.WaitGroup) {
	for i := 1; i <= 10; i++ {
		for true {
			if *count == 0 && mutex.TryLock() {
				break
			}
		}
		intChan <- i
		fmt.Printf("生产者生产了：%d\n", i)
		*count = 1

		mutex.Unlock()
	}
	close(intChan)
	defer waitGroup.Done() // 协程数计数器减一，即协程任务完成
}

func consumer(intChan chan int, mutex sync.Mutex, count *int, waitGroup *sync.WaitGroup) {
	for value := range intChan { // 遍历管道，当管道关闭之后会自动退出遍历
		for true {
			if *count == 1 && mutex.TryLock() {
				break
			}
		}
		fmt.Printf("消费者消费了：%d\n", value)
		*count = 0
		mutex.Unlock()

	}
	defer waitGroup.Done() // 协程数计数器减一，即协程任务完成
}

func main() {
	mutex := sync.Mutex{}

	waitGroup := sync.WaitGroup{}

	intChan := make(chan int, 10)
	count := 0

	waitGroup.Add(2) // 设置需要等待的协程数计数器

	go producer(intChan, mutex, &count, &waitGroup)
	go consumer(intChan, mutex, &count, &waitGroup)

	waitGroup.Wait() // 阻塞等待的协程数计数器归零

	fmt.Println("主程序执行完毕")
}
~~~

## 协程同步：管道实现生产者消费者有序生产消费，并且在主线程中通过sync.WaitGroup等待所有协程完成任务
~~~go
package main

import (
	"fmt"
	"sync"
)

func producer(intChan chan int, syncChan chan int, waitGroup *sync.WaitGroup) {
	for i := 1; i <= 10; i++ {
		intChan <- i
		fmt.Printf("生产者生产了：%d\n", i)
		<-syncChan
	}
	close(intChan)
	defer waitGroup.Done() // 协程数计数器减一，即协程任务完成
}

func consumer(intChan chan int, syncChan chan int, waitGroup *sync.WaitGroup) {
	for value := range intChan { // 遍历管道，当管道关闭之后会自动退出遍历
		fmt.Printf("消费者消费了：%d\n", value)
		syncChan <- 1
	}
	defer waitGroup.Done() // 协程数计数器减一，即协程任务完成
}

func main() {
	waitGroup := sync.WaitGroup{}

	intChan := make(chan int, 10)  // 存放生产消费数据的管道，如果容量设置为1就不需要在设置生产消费同步的管道了
	syncChan := make(chan int, 1)  // 生产消费同步管道，即生产一个然后消费一个的模式交替执行

	waitGroup.Add(2) // 设置需要等待的协程数计数器

	go producer(intChan, syncChan, &waitGroup)
	go consumer(intChan, syncChan, &waitGroup)

	waitGroup.Wait() // 阻塞等待的协程数计数器归零

	fmt.Println("主程序执行完毕")
}
~~~

## 协程同步：sync.Cond实现生产者消费者有序生产消费，并且在主线程中通过sync.WaitGroup等待所有协程完成任务
~~~go
package main

import (
	"fmt"
	"sync"
)

func producer(intChan chan int, cond *sync.Cond, waitGroup *sync.WaitGroup) {
	for i := 1; i <= 10; i++ {
		cond.L.Lock()
		intChan <- i
		fmt.Printf("生产者生产了：%d\n", i)
		cond.Wait() 
		cond.L.Unlock()
	}
	close(intChan)
	defer waitGroup.Done() // 协程数计数器减一，即协程任务完成
}

func consumer(intChan chan int, cond *sync.Cond, waitGroup *sync.WaitGroup) {
	for value := range intChan { // 遍历管道，当管道关闭之后会自动退出遍历
		cond.L.Lock()
		fmt.Printf("消费者消费了：%d\n", value)
		cond.L.Unlock()
		cond.Signal()  // 唤醒一个其他协程因为 cond.Wait() 调用造成等待
	}
	defer waitGroup.Done() // 协程数计数器减一，即协程任务完成
}

func main() {
	mu := &sync.Mutex{}
	cond := sync.NewCond(mu)

	waitGroup := sync.WaitGroup{}

	intChan := make(chan int, 10)

	waitGroup.Add(2) // 设置需要等待的协程数计数器

	go producer(intChan, cond, &waitGroup)
	go consumer(intChan, cond, &waitGroup)

	waitGroup.Wait() // 阻塞等待的协程数计数器归零

	fmt.Println("主程序执行完毕")
}
~~~

## 协程读写分离：sync.RWMutex读写锁操作map，并且在主线程中通过sync.WaitGroup等待所有协程完成任务
~~~go
package main

import (
	"fmt"
	"sync"
)

var m map[string]string

func readMap(a int, mu *sync.RWMutex, wg *sync.WaitGroup) {
	mu.RLock() // 读锁被占用的情况下会阻止写，不会阻止读，多个协程可以同时获得读锁
	for i := 0; i < 10; i++ {
        s, ok := m[string(i)]
        if ok {
		    fmt.Printf("%d号读协程读取到：%s\n", a, s)
        }
	}
	mu.RUnlock()
	defer wg.Done()
}

func writeMap(a int, mu *sync.RWMutex, wg *sync.WaitGroup) {
	mu.Lock() // 写锁会阻止其他协程对写锁和读锁的获取
	for i := 0; i < a; i++ {
		m[string(i)] = string(int('A') + i)
	}
	mu.Unlock()
	defer wg.Done()
}

func main() {
	mu := sync.RWMutex{}
	wg := sync.WaitGroup{}

	go writeMap(10, &mu, &wg) // 写操作

	for i := 0; i < 10; i++ { // 10个协程并发读
		wg.Add(1)
		go readMap(i, &mu, &wg)
	}
	wg.Wait()
}
~~~

## 协程安全map：sync.Map的功能和map除了协程安全之外没有区别，API使用方面有些不同
~~~go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var m sync.Map
	// 1. 写入
	m.Store("qcrao", 18)
	m.Store("stefno", 20)

	// 2. 读取
	age, _ := m.Load("qcrao")
	fmt.Println(age.(int))

	// 3. 遍历
	m.Range(func(key, value interface{}) bool {
		name := key.(string)
		age := value.(int)
		fmt.Println(name, age)
		return true
	})

	// 4. 删除
	m.Delete("qcrao")
	age, ok := m.Load("qcrao")
	fmt.Println(age, ok)

	// 5. 读取或写入
	m.LoadOrStore("stefno", 100)
	age, _ = m.Load("stefno")
	fmt.Println(age)
}
~~~

