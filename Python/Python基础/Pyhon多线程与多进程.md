# 多线程使用
### 使用多线程方式一：直接在创建线程对象时传递任务函数
~~~python
# 导入多线程包
from threading import Thread

def fn(name):
    for i in range(10):
        print(name, i)

if __name__ == '__main__':
    # 创建线程，target是任务函数，args是任务函数的参数（需要传递一个元组，注意元组只有一个参数时请在后面加上逗号否则无法识别元组）
    t = Thread(target=fn, args=("fn",))
    # 启动线程，并不是马上执行任务而是等待cpu的时间片分配后执行任务
    t.start()
    for i in range(10):
        print("main", i)

~~~
---


### 使用多线程方式二：直接在创建线程对象时传递任务函数
~~~python
# 导入多线程包
from threading import Thread

# 自定义线程类：继承Thread，重写run函数，run函数就是线程的任务函数
class MyThread(Thread):
    # self是任务函数的参数（需要传递一个元组，注意元组只有一个参数时请在后面加上逗号否则无法识别元组）
    # 在自定义线程类中一般是使用构造器传递任务函数的参数self
    def run(self):
        for i in range(10):
            print("fn", i)

if __name__ == '__main__':
    # 使用自定义线程类创建线程，
    t = MyThread()
    # 启动线程，并不是马上执行任务而是等待cpu的时间片分配后执行任务
    t.start()
    for i in range(10):
        print("main", i)

~~~
---


# 多进程使用
### 使用多线程方式一：直接在创建进程对象时传递任务函数
~~~python
# 导入多进程包
from multiprocessing import Process

def fn(name):
    for i in range(10):
        print(name, i)

if __name__ == '__main__':
    # 创建进程，target是任务函数, args是任务函数的参数（需要传递一个元组，注意元组只有一个参数时请在后面加上逗号否则无法识别元组）
    t = Process(target=fn, args=("fn"))
    # 启动进程，并不是马上执行任务而是等待cpu的时间片分配后执行任务
    t.start()
    for i in range(10):
        print("main", i)


~~~
---

### 使用多进程方式二：直接在创建进程对象时传递任务函数
~~~python
# 导入多进程包
from multiprocessing import Process

# 自定义进程类：继承Thread，重写run函数，run函数就是进程的任务函数
class MyProcess(Process):
    # self是任务函数的参数（需要传递一个元组，注意元组只有一个参数时请在后面加上逗号否则无法识别元组）
    # 在自定义进程类中一般是使用构造器传递任务函数的参数self
    def run(self):
        for i in range(10):
            print("fn", i)

if __name__ == '__main__':
    # 使用自定义进程类创建进程
    t = MyProcess()
    # 启动进程，并不是马上执行任务而是等待cpu的时间片分配后执行任务
    t.start()
    for i in range(10):
        print("main", i)

~~~
---

# 线程池和进程池使用（基本一致）
~~~python
# 导入线程池和进程池库，ThreadPoolExecutor是线程池、ProcessPoolExecutor是进程池
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def task(name):
    for i in range(10):
        print(name, i)


if __name__ == '__main__':
    with ThreadPoolExecutor(50) as t:
        # 提交10个任务给线程池
        for i in range(10):
            t.submit(task, f"任务{i}执行，打印：")
            
    # 等待线程池在的任务全部执行完后后面的代码才会继续执行
    print("123")

~~~
---