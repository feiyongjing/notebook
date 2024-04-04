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

# channel 通过通讯共享内存提供协程通信
- channel 的本质是一个队列，这个队列内部可以通过泛型设置放入多种数据类型的数据，并且线程安全的，
- channel 的方向有读、写、读写
- channel 多路复用
- channel 阻塞协程，有并发场景下的同步机制、并且关闭时发送广播的特性，作为协程退出通知
- channel 用于协程通讯，必须存在读写双方，否则会造成死锁


## 管道基本使用
~~~go
package main

import (
	"fmt"
	"sync"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func main() {
	// 管道必须make初始化之后才能存取数据，并且管道的容量一旦确定就无法修改了
    // 如果创建时没有声明管道的方向就代表该管道是可读可写的
	// 管道内部传输的数据类型是User 指针，管道的容量是10元素单位
	// 在没有协程的情况下，如果管道中没有数据了，再从管道读取数据会报 deadlock 错误
	ch := make(chan *User, 10)

	ch <- &User{Id: 1, Name: "张三", Age: 18}
	ch <- &User{Id: 2, Name: "李四", Age: 20}
	ch <- &User{Id: 3, Name: "王舞", Age: 22}

	fmt.Printf("ch管道中实际元素的数量是%v个，ch管道的容量是%v个元素单位\n", len(ch), cap(ch))

	user1 := <-ch
	fmt.Printf("%+v\n", user1)

	// 直接从管道取出数据，但是没有赋值给变量
	<-ch
	fmt.Printf("ch管道中实际元素的数量是%v个，ch管道的容量是%v个元素单位\n", len(ch), cap(ch))

    // 关闭管道
	close(ch)
	fmt.Printf("管道被关闭之后，无法继续向管道写数据，但是可以继续从管道读数据：%+v\n", <-ch)
	fmt.Printf("ch管道中实际元素的数量是%v个，ch管道的容量是%v个元素单位\n", len(ch), cap(ch))

    // 创建只写管道，只允许向管道写数据，不允许从管道读取数据
	ch1 := make(chan<- int, 10)

	// 创建只读管道，只允许从管道读取数据，不允许向管道写数据
	ch2 := make(<-chan int, 10)
}
~~~

## 管道的遍历
~~~go
package main

import (
	"fmt"
	"sync"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func main() {
	// 管道必须make初始化之后才能存取数据，并且管道的容量一旦确定就无法修改了
	// 管道内部传输的数据类型是User 指针，管道的容量是10元素单位
	// 在没有协程的情况下，如果管道中没有数据了，再从管道读取数据会报 deadlock 错误
	ch := make(chan *User, 10)

	ch <- &User{Id: 1, Name: "张三", Age: 18}
	ch <- &User{Id: 2, Name: "李四", Age: 20}
	ch <- &User{Id: 3, Name: "王舞", Age: 22}

	// 关闭管道，关闭管道的本质类似于文件内容的结束标志EOF
	close(ch)

	// 管道的遍历方式一：先取出管道的实际元素数量赋值给其他的遍历，再按照这个变量进行for循环
	// 注意一：提前赋值变量确定循环次数的原因是如果直接在循环中通过使用 len(ch) 获取管道元素的数量是变化的，并不能遍历整个管道中的元素
	// 注意二：一定要在管道关闭之后进行遍历，防止在其他协程继续提前取出管道中的元素，使管道元素数量减少，导致循环时需要取出的管道元素过多报 deadlock 错误
	//j := len(ch)
	//for i := 0; i < j; i++ {
	//	fmt.Printf("%+v\n", <-ch)
	//}

	// 管道的遍历方式二：for range
	// 管道在关闭之前进行遍历，会报 deadlock 错误，即读取到管道的全部数据之后还想再取数据
	// 管道在关闭之后进行遍历，会正常遍历管道，并且会在遍历完管道中的数据之后正常退出遍历
	for value := range ch {
		fmt.Printf("%+v\n", value)
	}

	// 管道的遍历方式三：for select
	// 使用for select遍历管道的好处是不用关闭管道也可以遍历，反而是关闭管道之后无法使用该方式进行遍历，并且遍历循环退出只能使用标签或使用return
	//testlabel:
	//	for {
	//		select {
	//		case value := <-ch:
	//			fmt.Printf("%+v\n", value)
	//		default:
	//			// 这里要退出循环只能使用标签，或者是在方法中可以使用return
	//			break testlabel
	//		}
	//	}
}
~~~

## 协程配合管道使用
~~~go
package main

import "fmt"

func readIntChan(intChan chan int, exitChan chan bool) {
	for true {
		i, ok := <-intChan
		if !ok {
			break
		}
		fmt.Printf("从管道读取到数据：%v\n", i)
	}

	exitChan <- true
    close(exitChan)
	fmt.Printf("从管道读取数据完毕\n")
}

func writeIntChan(intChan chan int) {
	for i := 1; i <= 50; i++ {
		intChan <- i
		fmt.Printf("向管道写入了数据：%v\n", i)
	}
	close(intChan)
}

func main() {
	intChan := make(chan int, 5)
	exitChan := make(chan bool, 1)

	// 向intChan管道写入50个数据，如果管道的容量满了会自动阻塞，而当管道被取出数据有空闲空间之后会继续写入数据
	go writeIntChan(intChan)

	// 从intChan管道读取数据直到管道没有数据可读，而管道又是打开状态就会阻塞，如果管道没有数据可读而管道又是关闭状态从管道取出数据会失败
	// 从intChan管道取出数据完毕之后，向exitChan管道写入数据，这样会通知主程序表示intChan管道读取完毕
	go readIntChan(intChan, exitChan)

	// 主程序通过exitChan管道阻塞，直到成功读取exitChan管道确定intChan管道数据读取完毕，然后主程序退出
	for true {
		_, ok := <-exitChan
		if ok {
			break
		}
	}
	fmt.Printf("主程序退出\n")
}
~~~

## 通过使用协程计算素数例子
~~~go
package main

import (
	"fmt"
	"math"
	"time"
)

// 初始化intChan管道，内部是需要计算是否是素数的数字列表
func initIntChan(intChan chan<- int) {
	for i := 1; i <= cap(intChan); i++ {
		intChan <- i
	}
	close(intChan)
}

// 从intChan管道取数据，计算是否是素数，如果是素数就就放入resChan管道
// 从intChan管道取不到数据就向exitChan管道写入数据，表示当前协程计算完毕
func primeNum(intChan <-chan int, resChan chan<- int, exitChan chan<- bool) {
	for true {
		num, ok := <-intChan
		if !ok {
			break
		}
		if isPrimeNum(num) {
			resChan <- num
		}
	}

	exitChan <- true
}

// 判断是否是素数，素数是只能被自己整除的数，但是1不是素数，2是素数
func isPrimeNum(num int) bool {
	for i := 2; i < int(math.Sqrt(float64(num))); i++ {
		if num%i == 0 {
			return false
		}
	}
	return num != 1
}

func main() {
	start := time.Now().Unix()

	size := 30000000
	intChan := make(chan int, size) // 存储需要计算是否是素数数据的管道
	resChan := make(chan int, size) // 计算结果管道，存储素数
	exitChan := make(chan bool, 30) // 协程退出判断管道

	go initIntChan(intChan)

	for i := 1; i <= cap(exitChan); i++ {
		go primeNum(intChan, resChan, exitChan)
	}

	go func() {
		for i := 1; i <= cap(exitChan); i++ {
			<-exitChan
		}

		end := time.Now().Unix()
		fmt.Printf("使用协程耗费的时间是：%d\n", end-start)
		close(resChan)
	}()

	// 如果需要查看使用协程耗费的时间请打开下面注释的代码
	// 会从resChan管道读取数据但是不会打印，打印耗费的时间比较多
	// 计算1到30000000内的素数，开启30个协程在10核心的cpu电脑上跑大约需要耗费15秒，但是打印素数大概需要耗费3分钟
	//for true {
	//	_, ok := <-resChan
	//	if !ok {
	//		break
	//	}
	//}

	for res := range resChan {
		fmt.Printf("素数是：%d\n", res)
	}

	fmt.Printf("主程序退出\n")
}
~~~

