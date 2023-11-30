# 线程
### 线程之间共享的东西
- 文件描述符表
- 各种信号的处理方式
- 当前的工作目录
- 用户ID和组ID
- 一些内存空间（不包含栈空间）

### 线程之间不共享的东西
- 线程ID
- 处理器现场和栈指针（内核栈）
- 独立的栈空间（用户空间栈）
- errno 变量
- 调度优先级
- 屏蔽的信号集

# 线程操作函数
## 注意使用以下函数时需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
## 调用 pthread_self 函数获取当前线程的ID
## 调用 pthread_create 函数进行线程的创建，man 3 pthread_create
## 调用 pthread_exit 函数进行线程的退出，man 3 pthread_exit
## 调用 pthread_join 函数手动进行指定线程的资源回收，man 3 pthread_join
## 调用 pthread_detach 函数设置指定线程在终止时自动进行资源回收，man 3 pthread_detach
## 调用 pthread_cancel 函数会取消线程，man 3 pthread_cancel
## 调用 pthread_testcancel 函数设置线程取消点，man 3 pthread_testcancel
~~~c
#include <pthread.h>

    // pthread_self 函数获取当前线程的ID
    // 返回值是线程ID
    pthread_t pthread_self(void);

    // pthread_create 函数进行线程的创建
    // thread 参数是存储线程ID的内存地址
    // attr 参数指向一个pthread_attr_t结构体，该结构体的内容在线程创建时用于确定新线程的属性（例如设置线程在终止时自动进行资源回收）。如果attr为NULL，则使用默认属性创建线程
    // start_routine 参数是线程的启动运行函数
    // arg 参数是线程执行启动运行函数是传递的参数
    // 创建成功返回值是0，创建失败需要使用 strerror 函数查看错误 
    int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                        void *(*start_routine) (void *), void *arg);

    // pthread_exit 函数进行线程的退出
    // 注意在线程中调用 exit 会导致进程退出（线程中的全部线程都会退出），请不要在线程中调用 exit 函数，而是使用pthread_exit 函数进行线程的退出
    // 使用pthread_exit 函数进行线程的退出，不会影响到同一进程内的其他线程（即使是主线程退出也不会影响到其他的线程）
    // retval 参数是线程的退出状态，可以自定义线程的退出状态，通常是 NULL，注意 retval 参数必须是堆上的空间地址，而不是栈上的内存地址，否则当线程退出时栈上的内存空间被回收而retval 参数分配的内存地址就成了野指针
    void pthread_exit(void *retval);

    // pthread_join 函数进行线程资源的回收
    // thread 参数是需要回收线程的ID
    // retval 参数是二级指针，最终指向的是线程的退出状态
    // 线程资源回收成功返回0，失败返回错误码
    // 注意在没有调用 pthread_detach 函数（或其他方式）设置指定线程在终止时自动进行资源回收 的情况下 pthread_join 会阻塞，但是一旦线程设置了终止时自动进行资源回收 pthread_join 将不会阻塞，而是立即返回错误（一般都是线程已经被回收）
    int pthread_join(pthread_t thread, void **retval);

    // pthread_detach 函数设置指定线程在终止时自动进行资源回收
    // 注意调用 pthread_detach 函数设置指定线程在终止时自动进行资源回收后，在调用 pthread_detach 函数进行手动回收线程资源会立即返回错误（一般都是线程已经被回收）
    // thread 参数是需要设置终止时自动进行资源回收的线程ID号
    // 设置成功返回0，失败返回错误码
    int pthread_detach(pthread_t thread);

    // pthread_cancel 函数取消线程，但是不是立即取消，而是当该线程指向代码到取消点时取消线程，一般的系统函数内部都有取消点，也可以使用 pthread_testcancel 函数 手动设置取消点
    // thread 参数线程的ID
    // 成功返回0，失败返回错误码
    int pthread_cancel(pthread_t thread);

    // pthread_testcancel 函数设置线程取消点，当调用pthread_cancel 指定取消的线程内部执行到 pthread_testcancel 函数后就会取消该线程
    // 成功返回0，失败返回错误码
    void pthread_testcancel(void);
~~~
---

### 线程创建、退出、回收代码例子如下
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~C
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

void * mythread(void * args){
    int n = *(int *) args;

    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    // 退出当前线程，从堆上申请空间设置线程退出的状态
    char* status = (char*) malloc(10);
    strcpy(status, "OK");
    pthread_exit(status);
}


int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%ld\n", getpid(), pthread_self());
    // 子线程的ID
    pthread_t thread;
    int n=100;
    int ret = pthread_create(&thread, NULL, mythread, &n);

    if(ret != 0){
        // 线程创建错误通过 strerror 函数查看
        printf("线程创建错误：%s\n", strerror(ret));
        return -1;
    }

    void *retval;

    // 阻塞设置等待线程回收
    pthread_join(thread, &retval);
    char* status = retval;
    printf("获取回收线程的退出状态是：%s\n", status);

    return 0;
}
~~~
---

### 线程创建时设置线程在终止时自动进行资源回收
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

void * mythread(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
}


int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%ld\n", getpid(), pthread_self());
    // 子线程的ID
    pthread_t thread;

    // 线程属性设置
    pthread_attr_t attr;
    // 初始化线程属性
    pthread_attr_init(&attr);
    // 设置线程属性为分离状态，PTHREAD_CREATE_JOINABLE是非分离属性
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);


    int n=100;
    int ret = pthread_create(&thread, NULL, mythread, &n);

    if(ret != 0){
        // 线程创建错误通过 strerror 函数查看
        printf("线程创建错误：%s\n", strerror(ret));
        return -1;
    }

    // 回收释放线程属性
    pthread_attr_destroy(&attr);

    // 睡眠防止进程退出而线程直接被回收，等待子线程执行完毕
    sleep(1);
    return 0;
}
~~~
---

### 循环创建多个线程代码例子如下
- 注意在使用pthread函数需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
- 注意多个线程的参数需要使用不同的内存空间，否则传递的参数将会是一样的参数
~~~C
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

void * mythread(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    //退出当前线程，从堆上申请空间设置线程退出的状态
    char* status = (char*) malloc(10);
    strcpy(status, "OK");
    pthread_exit(status);
}


int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%ld\n", getpid(), pthread_self());

    int ret;
    int i=5;
    pthread_t thread[5];
    int args[5];
    for(i=0; i<5; i++){
        args[i]=i;
        ret = pthread_create(&thread[i], NULL, mythread, &args[i]);

        if(ret != 0){
            // 线程创建错误通过 strerror 函数查看
            printf("线程创建错误：%s\n", strerror(ret));
            return -1;
        }
    }

    void *retval;
    for(i=0; i<5; i++){
        // 阻塞设置等待线程回收
        pthread_join(thread[i], &retval);
        char* status = retval;
        printf("获取回收线程的退出状态是：%s\n", status);
    }
    return 0;
}
~~~
---

# 线程安全互斥锁操作
## 常见的互斥锁初始化、回收互斥锁、阻塞加锁、非阻塞加锁、释放锁操作函数如下，使用man命令可以查看
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <pthread.h>

    // pthread_mutex_init 函数初始化互斥锁
    // mutex 参数是互斥锁的内存地址
    // attr 参数是初始化互斥锁的属性，通常默认是 NULL
    // 或者使用宏变量直接进行初始化 pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
    // 成功返回零，失败返回一个错误编号
    int pthread_mutex_init(pthread_mutex_t *restrict mutex,
                           const pthread_mutexattr_t *restrict attr);

    // pthread_mutex_destroy 函数回收互斥锁
    // mutex 参数是互斥锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_mutex_destroy(pthread_mutex_t * mutex);

    // pthread_mutex_lock 函数阻塞加互斥锁
    // mutex 参数是互斥锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_mutex_lock(pthread_mutex_t * mutex);

    // pthread_mutex_trylock 函数非阻塞加互斥锁
    // mutex 参数是互斥锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_mutex_trylock(pthread_mutex_t * mutex);

    // pthread_mutex_unlock 函数释放互斥锁
    // mutex 参数是互斥锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_mutex_unlock(pthread_mutex_t * mutex);
~~~
---

### 使用互斥锁保证线程数据安全例子，代码如下
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

#define NUM 8000
int number=0;

// 全局互斥锁
pthread_mutex_t mutex;

void * mythread(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    int i;
    int a;
    for(i=0; i<n;i++){
        pthread_mutex_lock(&mutex);
        a = number;
        a++;
        number=a;
        printf("线程间共享的全局变量值是：%d\n", number);
        pthread_mutex_unlock(&mutex);
    }
}

void * mythread1(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    int i;
    int a;
    for(i=0; i<n;i++){
        pthread_mutex_lock(&mutex);
        a = number;
        a++;
        number=a;
        printf("线程间共享的全局变量值是：%d\n", number);
        pthread_mutex_unlock(&mutex);
    }
}

int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%d\n", getpid(), pthread_self());

    // 互斥锁初始化
    pthread_mutex_init(&mutex, NULL);

    int ret;
    int i=5;
    pthread_t thread[5];
    int arg = NUM;
    for(i=0; i<5; i++){
        if(i%2==0){
            ret = pthread_create(&thread[i], NULL, mythread, &arg);
        }else{
            ret = pthread_create(&thread[i], NULL, mythread1, &arg);
        }
        if(ret != 0){
            // 线程创建错误通过 strerror 函数查看
            printf("线程创建错误：%s\n", strerror(ret));
            return -1;
        }

    }

    for(i=0;i<5;i++){
        // 阻塞设置等待线程回收
        pthread_join(thread[i], NULL);
    }
    // 回收互斥锁
    pthread_mutex_destroy(&mutex);

    return 0;
}
~~~
---

# 读写锁：本质是一把锁，只是线程进行的操作是读操作或写操作而已
### 同一时间只要有一个线程获取锁成功在进行读操作那么其他的线程就无法获取到锁进行写操作，但是其他线程可以获取锁进行读操作，即读操作共享锁
### 同一时间只要有一个线程获取锁成功在进行写操作那么其他的线程就无法获取到锁进行读写操作，即写操作独占锁
### 当读操作和写操作共同竞争锁时，写操作的优先级高于读操作的优先级
### 适用场景是读操作大于写操作的场景

## 读写锁常见操作函数如下，使用man命令可以查看
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <pthread.h>

    // pthread_rwlock_init 函数初始化读写锁
    // rwlock 参数是读写锁的内存地址
    // attr 参数是初始化读写锁的属性，通常默认是 NULL
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
                           const pthread_rwlockattr_t *restrict attr);

    // pthread_rwlock_destroy 函数回收读写锁
    // rwlock 参数是读写锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_destroy(pthread_rwlock_t * rwlock);

    // pthread_rwlock_rdlock 函数阻塞加读锁
    // rwlock 参数是读写锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_rdlock(pthread_rwlock_t * rwlock);

    // pthread_rwlock_wrlock 函数阻塞加写锁
    // rwlock 参数是读写锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_wrlock(pthread_rwlock_t * rwlock);

    // pthread_rwlock_tryrdlock 函数阻塞加读锁
    // rwlock 参数是读写锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_tryrdlock(pthread_rwlock_t * rwlock);

    // pthread_rwlock_trywrlock 函数阻塞加写锁
    // rwlock 参数是读写锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_trywrlock(pthread_rwlock_t * rwlock);

    // pthread_rwlock_unlock 函数释放读写锁
    // rwlock 参数是读写锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_rwlock_unlock(pthread_rwlock_t * rwlock);
~~~
---

### 使用读写锁保证线程数据安全例子，代码如下
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

int number=0;

// 全局读写锁
pthread_rwlock_t rwlock;

// 读线程回调函数
void * thread_read(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    while(1){
        pthread_rwlock_rdlock(&rwlock);
        printf("当前线程是读线程，线程号是%d，线程间共享的全局变量值是：%d\n", n, number);
        pthread_rwlock_unlock(&rwlock);
        sleep(1);
    }
}

// 写线程回调函数
void * thread_write(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    int a;
    while(1){
        pthread_rwlock_wrlock(&rwlock);
        a = number;
        a++;
        number=a;
        printf("当前线程是写线程，线程号是%d，线程间共享的全局变量值是：%d\n", n, number);
        pthread_rwlock_unlock(&rwlock);
        sleep(2);
    }
}

int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%d\n", getpid(), pthread_self());

    // 读写锁初始化
    pthread_rwlock_init(&rwlock, NULL);

    int ret;
    int i=0;
    pthread_t thread[8];

    for(i=0; i<8; i++){
        if(i<3){
            // 写线程创建
            ret = pthread_create(&thread[i], NULL, thread_write, &i);
        }else{
            // 读线程创建
            ret = pthread_create(&thread[i], NULL, thread_read, &i);
        }
        if(ret != 0){
            // 线程创建错误通过 strerror 函数查看
            printf("线程创建错误：%s\n", strerror(ret));
            return -1;
        }

    }

    for(i=0;i<8;i++){
        // 设置线程回收
        pthread_join(thread[i], NULL);
    }
    // 回收读写锁
    pthread_rwlock_destroy(&rwlock);

    return 0;
}
~~~
---

# 线程同步问题的解决方案
- 条件变量加互斥锁
- 信号量

## 线程同步方式：条件变量，常用函数如下
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <pthread.h>

    // pthread_cond_init 函数初始化条件变量
    // cond 参数是条件变量的内存地址
    // attr 参数是初始化互斥锁的属性，通常默认是 NULL
    // 或者使用宏变量直接进行初始化 pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
    // 成功返回零，失败返回一个错误编号
    int pthread_cond_init(pthread_cond_t *restrict cond,
              const pthread_condattr_t *restrict attr);
    
    // pthread_cond_destroy 函数回收条件变量
    // cond 参数是条件变量的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_cond_destroy(pthread_cond_t *cond);

    // pthread_cond_wait 函数阻塞当前线程并且等待其他线程唤醒，等待的过程中会释放互斥锁，唤醒需要持有互斥锁线程发送条件变量进行唤醒，当前线程被唤醒后将继续持有锁
    // cond 参数是条件变量的内存地址
    // mutex 参数是互斥锁的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_cond_wait(pthread_cond_t *restrict cond,
              pthread_mutex_t *restrict mutex);

    // pthread_cond_signal 函数唤醒一个阻塞在条件变量上的线程
    // cond 参数是条件变量的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_cond_signal(pthread_cond_t *cond);

    // pthread_cond_broadcast 函数唤醒其他所有阻塞在条件变量上的线程
    // cond 参数是条件变量的内存地址
    // 成功返回零，失败返回一个错误编号
    int pthread_cond_broadcast(pthread_cond_t *cond);
~~~
---

### 多线程使用条件变量实现线程同步，以下是生产者消费者模型例子代码
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>

// 链表
struct Node {
    int data;
    struct Node* next;
};

// 全局链表头节点
struct Node* head = NULL;

// 全局互斥锁
pthread_mutex_t mutex;

// 全局条件变量
pthread_cond_t cond;

// 生产者线程回调函数
void * producer(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    struct Node* pNode = NULL;
    while(1){
        pNode = (struct Node*) malloc(sizeof(struct Node));
        pNode->data = rand();
        // 添加互斥锁
        pthread_mutex_lock(&mutex);
        pNode->next = head;
        head = pNode;

        printf("当前线程是生产者线程，线程号是%d，生产的链表头节点是：%d\n", n, pNode->data);
        // 解除互斥锁
        pthread_mutex_unlock(&mutex);
        // 向其他线程发送条件变量，唤醒其中一个线程
        pthread_cond_signal(&cond);
        sleep(1);
    }
}

// 消费者线程回调函数
void * consumer(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    struct Node* pNode = NULL;
    while(1){
        // 添加互斥锁
        pthread_mutex_lock(&mutex);
        if(head == NULL){
            // 阻塞当前线程并且等待其他线程唤醒，等待的过程中会释放互斥锁，唤醒需要持有互斥锁线程发送条件变量进行唤醒，当前线程被唤醒后将继续持有锁
            pthread_cond_wait(&cond, &mutex);
        }
        int data= head->data;
        pNode = head->next;
        free(head);
        head = pNode;

        printf("当前线程是消费者线程，线程号是%d，消费了头链表节点：%d\n", n, data);
        // 解除互斥锁
        pthread_mutex_unlock(&mutex);
        sleep(2);
    }
}

int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%d\n", getpid(), pthread_self());

    // 互斥锁初始化
    pthread_mutex_init(&mutex, NULL);
    // 条件变量初始化
    pthread_cond_init(&cond, NULL);

    int ret;
    int i=0;
    pthread_t thread[8];

    for(i=0; i<8; i++){
        if(i<4){
            ret = pthread_create(&thread[i], NULL, producer, &i);
        }else{
            ret = pthread_create(&thread[i], NULL, consumer, &i);
        }
        if(ret != 0){
            // 线程创建错误通过 strerror 函数查看
            printf("线程创建错误：%s\n", strerror(ret));
            return -1;
        }
    }

    for(i=0;i<8;i++){
        // 设置线程回收
        pthread_join(thread[i], NULL);
    }

    // 回收互斥锁
    pthread_mutex_destroy(&mutex);
    // 回收条件变量
    pthread_cond_destroy(&cond);
    return 0;
}
~~~
---

## 线程同步方式：信号量，常用函数如下
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <semaphore.h>

    // sem_init 函数初始化信号量
    // sem 参数是信号量的内存地址
    // pshared 参数设置信号量是在 线程或进程中共享，0表示线程共享、1表示进程共享
    // value 参数是正整数，表示信号量的初始值。即代表初始有多少个线程可以同时执行同步的代码块，当信号量初始值为1时其实就是互斥锁，当信号量的初始值大于一时需要小心同步代码块中是否有操作共享变量，如果有操作共享变量可能会导致数据错乱、线程大量重复申请内存空间、线程重复释放内存空间等问题
    // 成功返回零，失败返回一个错误编号
    int sem_init(sem_t *sem, int pshared, unsigned int value);

    // sem_destroy 函数回收信号量
    // sem 参数是信号量的内存地址
    // 成功返回零，失败返回一个错误编号
    int sem_destroy(sem_t *sem);

    // sem_wait 函数执行会信号量减一，当信号量为零时会阻塞等待信号量
    // sem 参数是信号量的内存地址
    // 成功返回零，失败返回一个错误编号
    int sem_wait(sem_t *sem);

    // sem_trywait 函数执行会尝试信号量减一，该函数不会阻塞而是立即返回
    // sem 参数是信号量的内存地址
    // 成功返回零，失败返回-1
    int sem_trywait(sem_t *sem);

    // sem_post 函数执行会信号量加一
    // sem 参数是信号量的内存地址
    // 成功返回零，失败返回一个错误编号
    int sem_post(sem_t *sem);
~~~
---

### 多线程使用信号量实现线程同步，以下是生产者消费者模型例子代码
- 注意需要在编译时需要添加 -l pthread 参数进行链接 pthread 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
#include <semaphore.h>

struct Node {
    int data;
    struct Node* next;
};

struct Node* head = NULL;


// 生产者信号量
sem_t producer_sem;

// 消费者信号量
sem_t consumer_sem;

// 生产者线程回调函数
void * producer(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    struct Node* pNode = NULL;
    while(1){
        pNode = (struct Node*) malloc(sizeof(struct Node));
        pNode->data = rand();
        // 生产者信号量减一，当生产者信号量为零时会阻塞在这里
        sem_wait(&producer_sem);
        pNode->next = head;
        head = pNode;

        printf("当前线程是生产者线程，线程号是%d，生产的链表头节点是：%d\n", n, pNode->data);
        // 消费者信号量加一
        sem_post(&consumer_sem);
    }
}

// 消费者线程回调函数
void * consumer(void * args){
    int n = *(int *) args;
    printf("当前进程的id是：%d，当前子线程的id是：%ld, 子线程运行接收的参数是%d\n", getpid(), pthread_self(), n);
    struct Node* pNode = NULL;
    while(1){
        // 消费者信号量减一，当消费者信号量为零时会阻塞在这里
        sem_wait(&consumer_sem);
        int data= head->data;
        pNode = head;
        head = head->next;

        printf("当前线程是消费者线程，线程号是%d，消费了链表头节点：%d\n", n, data);
        // 生产者信号量加一
        sem_post(&producer_sem);
        // 回收堆空间
        free(pNode);
        pNode = NULL;
    }
}

int main(){
    printf("当前进程的id是：%d，当前主线程的id是：%ld\n", getpid(), pthread_self());

    // 生产者信号量初始化，注意这里初始化信号量的值为 1 表示互斥锁，如果大于1会出现线程间共享变量可能会导致数据错乱、线程大量申请堆内存空间、线程重复释放堆内存空间等问题
    sem_init(&producer_sem, 0, 1);
    // 消费者信号量初始化
    sem_init(&consumer_sem, 0, 0);

    int ret;
    int i=0;
    pthread_t thread[8];

    for(i=0; i<8; i++){
        if(i<4){
            ret = pthread_create(&thread[i], NULL, producer, &i);
        }else{
            ret = pthread_create(&thread[i], NULL, consumer, &i);
        }
        if(ret != 0){
            // 线程创建错误通过 strerror 函数查看
            printf("线程创建错误：%s\n", strerror(ret));
            return -1;
        }
    }

    for(i=0;i<8;i++){
        // 设置线程回收
        pthread_join(thread[i], NULL);
    }

    // 回收生产者信号量
    sem_destroy(&producer_sem);
    // 回收消费者信号量
    sem_destroy(&consumer_sem);
    return 0;
}
~~~
---