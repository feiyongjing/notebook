### 守护进程与一般进程的区别
- 独立于终端，即终端关闭不会关闭守护进程
- 用户的登录和退出不会影响到守护进程
- 提供某种服务进行周期性的执行任务
- 一般ps aux 查看进程，进程名称都是以 d 结尾，例如 mysqld、dockerd、influxd 等

# 进程组、会话与守护进程

### 进程组
- 进程组：每个进程都属于一个进程组，进程组是一个或多个进程的集合体，它们共同完成一个实体任务，每个进程都有一个进程组长，默认进程组的ID与进程组长的PID相同
- 进程组ID = 进程组长的PID = 第一个创建的进程PID，例如一个父进程创建多个子进程，那么进程组ID = 父进程ID
- 进程组的生存期：只要进程组中的进程有一个存活那么进程组就存在，与进程组长是否终止无关

### 会话
- 会话是一个或者多个进程组的集合
- 创建会话需要root权限（Ubuntu 不需要），创建会话的进程不能是进程组的组长，但是创建会话的进程在创建会话后会成为一个进程组的组长同时也是会话的会长
- 新建会话的流程：父进程先调用 fork 函数创建子进程，然后父进程终止，最后子进程调用 setsid 函数创建会话
- ps ajx 查看会话和进程组ID，SID列就是会话ID，PGID列就是进程组ID

### 成为会话会长的进程脱离了控制终端，同时这个进程也被称为**守护进程**

### setsid 函数创建会话
~~~c
#include <sys/types.h>
#include <unistd.h>

    // 创建会话，返回会话ID，返回-1表示会话创建失败
    pid_t setsid(void);

~~~
---

### 以下是创建守护进程例子，守护进程在后台周期性的打印当前时间到文件中，当然也有些待优化的地方，比如文件频繁的打开关闭，文件内容过多的问题。
- 注意实验完毕后 kill -9 杀死子进程避免不停的打印内容到文件导致占用大量磁盘空间
~~~c
#include <sys/types.h>
#include <sys/wait.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <time.h>
#include <sys/time.h>
#include <fcntl.h>

void sighandler(int signo){
    printf("捕获的信号编号：%d\n", signo);
    // 打开文件写入当前时间到文件末尾
    int fd = open("./test.log", O_RDWR | O_APPEND | O_CREAT, 0755);
    if( fd<0 ){
        perror("文件打开失败");
        return ;
    }
    time_t t;
    time(&t);
    char *p = ctime(&t);

    write(fd, p, strlen(p));

    close(fd);
    return;
}

int main(){

    printf("父进程的pid是%d\n", getpid());

    pid_t p;
    int i;
    p=fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p==0){
        printf("子进程的pid是%d\n", getpid());
        // 等待父进程死亡
        sleep(3);

        // 创建会话
        setsid();

        // 改变工作目录，防止工作目录所在的设备被物理拔出导致进程退出
        chdir("/root/test1");

        // 改变工作目录的umask权限，请根据进程的核心功能赋予对应的权限，这里给的最大权限
        umask(0000);

        // 关闭控制台的标准输入、标准输出、标准错误，即脱离控制台的影响
        close(STDIN_FILENO);
        close(STDOUT_FILENO);
        close(STDERR_FILENO);

        // 注册SIGALRM信号捕获函数
        signal(SIGALRM, sighandler);

        // 设置时钟定时发送SIGALRM信号
        struct itimerval tm;
        tm.it_interval.tv_sec=1;
        tm.it_interval.tv_usec=0;
        tm.it_value.tv_sec=3;
        tm.it_value.tv_usec=0;

        setitimer(ITIMER_REAL, &tm, NULL);

        while(1){
            sleep(1);
        }
    } 
    return 0;
}
~~~
---


