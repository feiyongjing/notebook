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

## sync.Pool临时缓存对象
~~~go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

// 用来统计实例真正创建的次数
var numCalcsCreated int32

// 创建实例的函数
func createBuffer() interface{} {
	// 这里要注意下，非常重要的一点。这里必须使用原子加，不然有并发问题；
	atomic.AddInt32(&numCalcsCreated, 1)
	buffer := make([]byte, 1024)
	return &buffer
}

func main() {
	// sync.Pool中用于缓存临时对象，但是缓存对象在不确定的时间会被自动被回收掉
	// 创建sync.Pool，New设置Poll中存储对象的创建函数
	// 取出缓存对象时，如果有缓存对象就直接取出
	// 如果没有缓存的对象，即缓存对象在不确定的时间被回收掉了会通过New设置创建出一个新的对象
	// 注意Pool本身是线程安全的，但是New属性函数创建函数对象不是线程安全的
	bufferPool := &sync.Pool{
		New: createBuffer,
	}

	// 多协程并发测试
	numWorkers := 1024 * 1024
	wg := sync.WaitGroup{}
	wg.Add(numWorkers)

	for i := 0; i < numWorkers; i++ {
		go func() {
			defer wg.Done()
			// 向Pool中获取一个缓存的对象，如果没有就通过New属性函数创建一个返回
			buffer := bufferPool.Get()
			_ = buffer.(*[]byte)
			// 向Pool中放入一个缓存的对象
			defer bufferPool.Put(buffer)
		}()
	}
	wg.Wait()
	fmt.Printf("Pool中的缓存对象创建了 %d 次\n", numCalcsCreated)
}
~~~

## 通过sync.Once实现单例模式
~~~go
package main

import (
	"fmt"
	"sync"
)

type User struct {
	Name string
	Age  int
}

var (
	singletonUser *User
	once          sync.Once
)

func GetSingletonUser() *User {
	// sync.Once的Do函数设置传入的函数参数在多协程中只保证执行一次
	once.Do(func() {
		fmt.Println("单例User对象初始化只执行一次")
		singletonUser = &User{}
	})
	return singletonUser
}

func main() {
	wg := sync.WaitGroup{}

	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			s := GetSingletonUser()
			fmt.Printf("单例User对象的指针地址: %p\n", s)
		}()
	}

	wg.Wait()
}
~~~

# 协程原子操作
- sync/atomic提供了5种类型的原子操作和1个Value类型
## 5种原子操作
~~~go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var newValue1 int32 = 200
	var dst1 int32 = 100
	// atomic.SwapXXX函数，XXX表示一些常见的类型
	// 这些函数可以协程安全（原子操作）的修改指定类型的指针，使其指向新的值，并且函数返回原来的值
	oldValue1 := atomic.SwapInt32(&dst1, newValue1)
	// 打印结果
	fmt.Println("old value: ", oldValue1, " new value:", dst1)

	var dst2 int32 = 100
	var oldValue2 int32 = 100
	var newValue2 int32 = 200
	// atomic.CompareAndSwapXXX函数，XXX表示一些常见的类型
	// 这些函数可以协程安全（原子操作）的先比较指针指向值和指定的old值是否相等，如果相等，则指针指向new值,并且返回true。如果不相等则直接返回false
	swapped := atomic.CompareAndSwapInt32(&dst2, oldValue2, newValue2)
	// 打印结果
	fmt.Printf("old value: %d, swapped value: %d, swapped success: %v\n", oldValue2, dst2, swapped)

	var a int32 = 100
	var b int32 = 200
	// atomic.AddXXX函数，XXX表示一些常见的类型
	// 这些函数可以协程安全（原子操作）的将指针指向值和指定的值进行相加，返回相加的结果
	sum := atomic.AddInt32(&a, b)
	fmt.Printf("a value: %d, b value: %d, sum result: %v\n", a, b, sum)

	var res1 int32 = 100
	// atomic.LoadXXX函数，XXX表示一些常见的类型
	// 这些函数可以协程安全（原子操作）的获取指针指向的值
	result1 := atomic.LoadInt32(&res1)
	fmt.Println("result=", result1)

	var val1 int32 = 100
	var newValue int32 = 200
	// atomic.StoreXXX函数，XXX表示一些常见的类型
	// 这些函数可以协程安全（原子操作）的修改指针指向的值使其指向指定的值
	atomic.StoreInt32(&val1, newValue)
	fmt.Println("result=", val1)
}
~~~

## Vlaue类型设置任意类型的原子操作
~~~go
package main

import (
	"fmt"
	"sync/atomic"
)

func loadConfig() map[string]string {
	m := make(map[string]string)
	m["张三"] = "十八岁"
	return m
}

func main() {
	// config变量用来存放该服务的配置信息
	config := atomic.Value{}
	// Store(）函数设置任意类型的值
	config.Store(loadConfig())

	// Load() 函数获取值，返回的是一个interface{}类型，所以要先强制转换一下
	m := config.Load().(map[string]string)

	// CompareAndSwap 函数可以比较值是否相等，相等再进行修改，并且返回true，否则返回false
	// Swap 函数可以修改值并且返回旧的值
	
	fmt.Printf("原子操作指定的值：%v", m)
}
~~~
