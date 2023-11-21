# 管道
- 管道的本质是一块内核缓冲区，内部是环形队列实现的
- 管道有两个文件描述符引用，一个表示读端，另一个表示写端，规定数据从写端流入管道，从读端流出
- 管道的数据只能单向流通从读端到写端，如果需要双向流动则需要再开辟一个管道反向流通数据
- 当管道两端的进程都终结时，管道也会自动消失
- 管道的读和写默认都是阻塞的，可以通过 fcntl() 函数手动设置读写端管道文件描述符，添加 O_NONBLOCK 就变成非阻塞
- 管道中的数据一旦读走，管道中就不再有读走的数据，即不可反复读取
- 默认的管道缓冲区是4k，使用ulimit -a 可以查看。如果管道的缓冲区满了再进行写入会直接阻塞
- 管道的读端如果全部关闭（多个进程的读端）后进行写操作，则管道破碎，进程终止，内核给当前进程发送SIGPIPE信号
- 管道的写端如果全部关闭（多个进程的写端）后进行读操作，同时管道没有数据则读操作不阻塞直接返回0，有数据还是会阻塞

# 匿名管道（PIPE）和命名管道（FIFO）
- 匿名管道只能用于有血缘关系的进程通信，而命名管道可以用于没有血缘关系的进程通信
- 命名管道（FIFO）是Linux中的一种基础文件类型（使用ls -l 查看文件类型是p的文件就是命名管道FIFO），命名管道FIFO文件在磁盘上没有数据块，文件大小为0，仅仅用于表示内核中的一条通道，进程对这个文件的读写其实是在操作内核缓冲区，这样就做到了进程通信。命名管道FIFO严格遵循先进先出原则，总是从开始处读取数据而从末尾写入数据，并且不支持lseek()文件定位等操作，但是其他普通文件的open()、read()、write()、close()等函数都可以进行调用操作

## 匿名管道创建函数
查看命令：man 2 pipe

~~~C
#include <unistd.h>

    int pipe(int pipefd[2]);
    
#define _GNU_SOURCE /* See feature_test_macros(7) */
#include <fcntl.h> /* Obtain O_* constant definitions */
#include <unistd.h>

    int pipe2(int pipefd[2], int flags);
    
// pipe函数创建管道
// pipefd[2] 参数是两个长度的数组，第一个元素是管道的读端文件描述符、第二个元素是管道的写端文件描述符
// pipe函数返回值为0表示成功创建管道，返回-1表示失败并设置了 error 值
// 其实pipefd[2]的两个文件描述符在函数调用后，它们指向的是内核的缓冲区

~~~
---
 
## 父子进程使用匿名管道通信
1. 创建管道
2. 创建子进程
3. 父进程关闭管道的一端，子进程关闭管道的另一端
4. 父子进程分别向管道读写数据
~~~c
#include <sys/types.h>  
#include <sys/wait.h>  
#include <dirent.h>  
#include <stdio.h>  
#include <string.h>  
#include <unistd.h>  

int main(){  

    pid_t p;
    // 管道的读端文件描述符和写端文件描述符存储  
    int fd[2];
    // 创建管道  
    int ret = pipe(fd);
    
    if(ret<0){
        perror("创建管道失败\n");
        return -1;
    }

    p = fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p>0){
        printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());
        // 父进程关闭管道的读端  
        close(fd[0]);

        printf("父进程第一次向管道写入数据\n");
        char hello[100] = "hello world";
        write(fd[1], hello, strlen(hello));

        sleep(5);
        printf("父进程第二次向管道写入数据\n");
        char hello1[100] = "hello world1";
        write(fd[1], hello1, strlen(hello1));
        
        // wait函数阻塞等待子进程结束运行，然后回收子进程资源
        wait(NULL);
    }

    if(p==0){
        printf("子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
        // 子进程关闭管道的写端  
        close(fd[1]);

        char buf[100];
        // 管道文件默认是阻塞的，所以会等待管道写入数据后再读取数据  
        read(fd[0], buf, sizeof(buf));
        printf("子进程从管道第一次读取的数据是：%s\n", buf);

        char buf1[100];
        read(fd[0], buf1, sizeof(buf1));
        printf("子进程从管道第二次读取的数据是：%s\n", buf1);
        return 88;
    }

    return 0;
}
~~~
---
 
## 使用匿名管道实现父子进程的 ps aux | grep bash
~~~C
#include <sys/types.h>
#include <sys/wait.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){

    pid_t p;
    int fd[2];
    // 创建管道
    int ret = pipe(fd);

    p = fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p>0){
        printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());
        // 父进程关闭管道的读端
        close(fd[0]);

        // 使标准输出指向管道的写端
        dup2(fd[1], STDOUT_FILENO);
        execlp("ps", "ps", "aux", NULL);
    }

    if(p==0){
        printf("子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
        // 子进程关闭管道的写端
        close(fd[1]);

        // 使标准输入指向管道的读端
        dup2(fd[0], 0);
        //dup2(fd[0], STDIN_FILNO);
        execlp("grep", "grep", "--color=auto", "bash", NULL);
    }

    return 0;
}
~~~
---

## 命名管道FIFO创建
### 方式一：使用命令创建 
~~~shell
mkfifo [管道名称]
~~~
---
### 方式二：使用系统函数 mkfifo() 创建 
以下是两段代码示例展示了两个毫无关系的进程通过命名管道FIFO通信，请先启动读端的代码再启动写端的代码，否则读取的数据会有一些错乱的数据（有时会读取到内核缓冲区中已经读取过的数据，解决方案应该是读端读取数据时按行读取数据）
1. 写端代码
~~~c

#include <sys/types.h>
#include <sys/wait.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>

int main(){

    // 判断命名管道FIFO文件是否已经存在
    int fd = access("./test-fifo", F_OK);
    if( fd!=0 ){

        // 若命名管道FIFO文件不存在就创建
        int fd = mkfifo("./test-fifo", 0777);
        if( fd<0 ){
            perror("命名管道文件创建失败");
            return -1;
        }

    }

    fd = open("./test-fifo", O_RDWR);
    if( fd<0 ){
        perror("命名管道文件打开失败");
        return -1;
    }
    char hello[100];

    int i=0;
    while(1){
        i++;
        memset(hello, 0x00, sizeof(hello));
        sprintf(hello, "%s:%d\n", "hello world", i);
        write(fd, hello, strlen(hello));
        sleep(1);
    }

    close(fd);
    return 0;
}
~~~
---

2. 读端代码
~~~c
#include <sys/types.h>
#include <sys/wait.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h>

int main(){

    int fd = open("./test-fifo", O_RDWR);
    if( fd<0 ){
        perror("命名管道文件打开失败");
        return -1;
    }
    char buf[100];

    //char c;
    //int i=0;
    while(1){
        read(fd, buf, sizeof(buf));
        printf("从命名管道读取的数据是：%s\n", buf);

        // 以下是按行读取数据，可以解决先启动写端的代码，再读端的代码的数据错乱问题
        //while (read(fd, &c, 1) > 0) {
        //    if (c == '\n') {
        //        buf[i] = '\0';
        //        printf("从命名管道读取的数据是：%s\n", buf);
        //        i = 0;
        //    } else {
        //        buf[i++] = c;
        //    }
        //}
    }
    close(fd);

    return 0;
}
~~~
---


## 手动设置管道读写端为非阻塞
### 将读端设置为非阻塞
    - 如果管道中有数据，无论写端是否关闭，读端都会读取到管道内部的数据
    - 如果管道中没有数据，且写段没有关闭，读端读取数据会返回-1
    - 如果管道中没有数据，且写段关闭，读端读取数据会返回0
~~~C
#include <sys/types.h>
#include <dirent.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main(){
    int fd[2];
    // 创建管道
    int ret = pipe(fd);

    // 添加一个新的flag（O_NONBLOCK将管道文件变为非阻塞），不会破坏原有的flags
    // 这里操作的是读端
    int flags = fcntl(fd[0], F_GETFL, 0);
    fcntl(fd[0], F_SETFL, flags | O_NONBLOCK);
}
~~~
---

## 查看管道的缓冲区大小
### 一般默认的管道大小可以使用 ulimit -a 命令查看，默认是4KB。也可以使用系统函数来查看指定的管道大小
~~~c
#include <sys/types.h>
#include <dirent.h>
#include <stdio.h>
#include <unistd.h>

int main(){
    int fd[2];
    int ret = pipe(fd);

    // fpathconf() 函数查看管道的大小，单位是byte
    // 注意第一参数是管道的文件描述符，第二参数是必须是 _PC_PIPE_BUF 表示查看匿名管道PIPE或者是命名管道FIFO的大小
    printf("pipe size =%ld\n", fpathconf(fd[0], _PC_PIPE_BUF));
}
~~~
---
