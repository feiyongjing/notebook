
## context 接口函数
~~~go
type Context {
    // 返回 Context 的截止时间，表示在这个时间点之后，Context 会被自动取消。
    // 如果 Context 没有设置截止时间，该方法返回一个零值 time.Time 和 false
    Deadline() (deadline time.Time, ok bool)
    
    // 返回一个只读管道，当 Context 被取消时，该管道会被关闭。你可以通过监听这个管道来检测 Context 是否被取消。
    // 如果 Context 永不取消，则返回 nil
    Done() <-chan struct{}
    
    // 返回一个 error 值，表示 Context 被取消时产生的错误。如果 Context 尚未取消，该方法返回 nil。
    Err() error
    
    // 返回与 Context 关联的键值对，一般用于在协程之间传递请求范围内的信息。如果没有关联的值，则返回 nil。
    Value(key interface{}) interface{}
}
~~~

## 不同的context上下文实现介绍，参考：https://juejin.cn/post/7233981178101186619
1. context.Background() 函数返回一个非 nil 的空 Context，它没有携带任何的值，也没有取消和超时信号。通常作为根 Context 使用
2. context.TODO() 函数返回一个非 nil 的空 Context，它没有携带任何的值，也没有取消和超时信号。虽然它的返回结果和 context.Background() 函数一样，但是它们的使用场景是不一样的，如果不确定使用哪个上下文时，可以使用 context.TODO()
3. context.WithValue(parent Context, key, val any) 函数接收一个父 Context 和一个键值对 key、val，返回一个新的子 Context，并在其中添加一个 key-value 数据对。返回的子Context内部存储了一对key-value和父Context，内部重写了Value函数，在子Context中调用Value获取key的value时如果没有会去父Context中查找
4. context.WithCancel(parent Context) (ctx Context, cancel CancelFunc) 函数接收一个父 Context，返回一个新的子 Context 和一个取消函数，当取消函数被调用时，子 Context 会被取消，同时会向子 Context 关联的 Done() 通道发送取消信号，届时其衍生的子孙 Context 都会被取消。这个函数适用于手动取消操作的场景。
5. context.WithCancelCause(parent Context) (ctx Context, cancel CancelCauseFunc) 函数是 Go 1.20 版本才新增的，其功能类似于 context.WithCancel()，但是它可以设置额外的取消原因，也就是 error 信息，返回的取消函数被调用时，需传入一个 error 参数。
6. context.WithDeadline(parent Context, d time.Time) (Context, CancelFunc) 函数接收一个父 Context 和一个截止时间作为参数，返回一个新的子 Context。当截止时间到达时，子 Context 其衍生的子孙 Context 会被自动取消。这个函数适用于需要在特定时间点取消操作的场景。
7. context.WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) 函数和 context.WithDeadline() 函数的功能是一样的，其底层会调用 WithDeadline() 函数，只不过其第二个参数接收的是一个超时时间，而不是截止时间。这个函数适用于需要在一段时间后取消操作的场景。

### 通过 context.WithValue 进行协程信息传递例子
~~~go
package main

import (
	"context"
	"fmt"
	"sync"
)

func f1(ctx context.Context, wg *sync.WaitGroup) {
	go f2(context.WithValue(ctx, "李四", "20岁"), wg)
}

func f2(ctx context.Context, wg *sync.WaitGroup) {
	fmt.Printf("在context链路中获取张三的年龄%s\n", ctx.Value("张三"))
	wg.Done()
}

func main() {
	ctx := context.WithValue(context.Background(), "张三", "18岁")
	wg := sync.WaitGroup{}
	wg.Add(1)
	go f1(ctx, &wg)
	wg.Wait()
}
~~~

### 通过 context.WithCancel 进行控制协程退出协程传递例子
~~~go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

func f1(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("下班啦...")
			return
		default:
			fmt.Println("正在工作中...")
		}
	}
}

func main() {
	ctx, cancelFunc := context.WithCancel(context.Background())

	go f1(ctx)

	// 等待协程执行
	time.Sleep(time.Second * 1)
	// 调用取消函数，会关闭上下文中的管道，通过关闭管道控制协程退出
	cancelFunc()
}
~~~

### 通过 context.WithTimeout 进行控制协程执行时间，执行时间结束之后协程会退出例子
~~~go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// 使用 WithTimeout 创建一个带有超时的上下文对象
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	// 在另一个协程中执行耗时操作
	go func() {
		// 模拟一个耗时的操作，例如数据库查询
		time.Sleep(5 * time.Second)
		cancel()
	}()

	// 通过上下文管道等待协程退出
	select {
	case <-ctx.Done():
		fmt.Println("操作已超时")
	case <-time.After(10 * time.Second):
		fmt.Println("操作完成")
	}
}
~~~




