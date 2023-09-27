### 协程的作用是在执行函数A时可以随时中断去执行函数B，然后中断函数B继续执行函数A（可以自由切换）。但这一过程并不是函数调用，这一整个过程看似像多线程，然而协程只有一个线程执行。协程是运行在单线程当中的"并发"，协程相比多线程一大优势就是省去了多线程之间的切换开销，获得了更大的运行效率。协程可以处理IO密集型程序的效率问题，但是处理CPU密集型不是它的长处，如要充分发挥CPU利用率可以结合多进程+协程。协程不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在控制共享资源时也不需要加锁，因此执行效率高很多。

# 使用协程
~~~python
# 导入异步IO库
import asyncio
# 时间库
import time

import platform

# await 关键字只能在异步函数中使用，用于等待另一个异步函数或异步操作完成。
# await 关键字的作用是将异步操作的结果返回给调用者。当使用 await 关键字等待一个异步操作时，当前协程会被挂起，直到异步操作完成并返回结果，然后再继续执行协程的后续代码。在等待异步操作的过程中，事件循环可以继续执行其他协程，从而实现并发执行。
# async关键字定义异步函数和异步上下文管理器，async关键字声明的函数是协程函数，直接执行协程函数并不会执行函数内的代码而是返回一个协程对象
# 想要执行函数中的代码请使用asyncio.run()函数，并将协程对象直接传入或者是封装多个协程对象成task对象元组传入
# 任务：当一个协程通过 asyncio.create_task() 等函数被封装返回 Task 对象，并调度其执行

async def fn1(name):
    print(name)
    # await 声明协程异步操作，当执行到这里时会进行异步切换到其他的任务上
    await asyncio.sleep(2)
    print(name)


async def fn2(name):
    await asyncio.sleep(3)
    print(name)


async def fn3(name):
    await asyncio.sleep(4)
    print(name)


async def main():
    # 方式一：手动设置任务列表
    tasks = [
        asyncio.create_task(fn1("1")),
        asyncio.create_task(fn2("2")),
        asyncio.create_task(fn3("3"))
    ]
    await asyncio.wait(tasks)

    # 方式二：asyncio.TaskGroup()设置任务列表
    # async with asyncio.TaskGroup() as tg:
    #     task1 = tg.create_task(fn1("1"))
    #     task2 = tg.create_task(fn2("2"))
    #     task3 = tg.create_task(fn3("3"))


if __name__ == '__main__':
    # 检查平台，windows平台需要注意设置事件循环策略
    if platform.system() == 'Windows':
        asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
    t1 = time.time()
    asyncio.run(main())
    t2 = time.time()
    print("一共使用了",t2-t1)
~~~
---