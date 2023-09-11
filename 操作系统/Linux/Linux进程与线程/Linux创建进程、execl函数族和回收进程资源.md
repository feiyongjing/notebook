# 创建进程：使用fork()函数
查看方式：man fork
~~~C
#include <unistd.h>
pid_t fork(void); 

//调用fork函数会以当前进程作为父进程，根据父进程模板创建一个子进程。
//子进程的用户区父进程的用户完全一样，而内核区有些不一样，例如pid不一样。
//在父进程中fork函数返回的是子进程的pid，这个pid一定大于0
//在子进程中fork函数返回的是0，并且子进程只会执行fork函数之后的代码，而不会执行fork之前的代码，当然一些需要使用的变量（包含全局变量）是从父进程的用户区复制（深拷贝）的，所以进程之间相同变量名的变量实际是存储在不同的虚拟内存空间中，在其中一个进程中修改变量不会影响其他进程的的变量
//但是其实不同进程的同名变量的虚拟内存地址其实是在变量不发生修改时是指向同一块物理内存地址，一旦其中一个进程发生修改会先在物理内存中开辟一块空间存放修改后的值，然后将修改后数据存放的物理内存地址放入该进程的用户区。即所有进程在不修改的情况下读取的是同一块物理内存地址所在的数据，而发生修改则开辟该进程独有的物理内存地址存放修改的数据

//fork函数返回值小于0表示进程创建失败

//也就是说父进程会执行fork函数返回值大于0的一些逻辑代码，而子进程执行fork函数返回值等于0的一些逻辑代码
//父进程和子进程的执行顺序是它们谁先得到cpu的时间片，谁就先执行

//父进程和子进程是共享内核区中pcb区域的文件描述符表，但是只要父子进程只要有一个没有退出那么全部打开的文件就没有关闭

//进程退出时会进程会回收自己的用户区空间
//当子进程先于父进程执行完退出，但是父进程没有调用wait/waitpid回收子进程内核区的PCB资源，则子进程变成僵尸进程
//如果父进程先执行完退出，这个子进程被称为孤儿进程。但是子进程会被pid是1的进程领养，pid是1的父进程可以回收子进程内核区的PCB资源

//使用kill -9 并不能杀死僵尸进程，原因是僵尸进程本身就已经是终止状态了，
//解决僵尸进程的方式是让它的父进程调用wait/waitpid回收僵尸进程占用的资源，或者是直接 kill -9 杀死僵尸进程的父进程让僵尸进程被pid是1的进程领养，pid是1的进程会自动的回收僵尸进程

~~~
---

## 创建子进程协调父进程执行代码例子
~~~C
#include <sys/types.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){

    printf("只有父进程执行的逻辑代码，父进程的pid是%d\n", getpid());

    pid_t p;
    p=fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p>0){
        printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());
    }

    if(p==0){
        // sleep(3); 睡眠会导致父进程先执行完，然后子进程将pid为1的进程作为父进程
        printf("只有子进程执行的逻辑代码，子进程的pid是%d，子进程的父进程pid是%d\n", getpid(), getppid());
    }

    printf("父进程和子进程都执行的逻辑代码，执行代码的进程pid是%d\n", getpid());

    return 0;
}
~~~
---

## 循环创建指定数量的子进程
~~~C
#include <sys/types.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){
    
    printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());

    pid_t p;
    int i;
    for(i=0; i<3; i++){
        p=fork();
        if(p<0){
            perror("创建子进程失败\n");
            return -1;
        }

        if(p==0){
            // 注意跳出循环，否则就不是创建循环指定子进程数量
            break;
        }
    }

    if(i==0){
        printf("第一个子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
    }

    if(i==1){
        printf("第二个子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
    }

    if(i==2){
        printf("第三个子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
    }
    
    if(i==3){
        printf("父进程执行的逻辑代码，父进程的pid是%d\n", getpid();
    }

    return 0;
}
~~~
---
 

# execl()函数族
查看方式：man execl
~~~C
#include <unistd.h>

   extern char **environ;
   
   int execl(const char *path, const char *arg, ...);
   int execlp(const char *file, const char *arg, ...);
   int execle(const char *path, const char *arg, ..., char * const envp[]);
   int execv(const char *path, char *const argv[]);
   int execvp(const char *file, char *const argv[]);
   int execvpe(const char *file, char *const argv[], char *const envp[]);
   
// execl()函数族的作用是在当前的进程中执行其他的程序，注意并没有创建新的进程   

// execl() 一般是用于让进程执行一些自己编写的程序
// execlp() 一般是用于让进程执行一些系统程序

// 参数
// path 是要执行程序的绝对路径，
// file是要执行程序（没有写绝对路径，而是在Path环境变量中去寻找可执行程序）
// arg 是变长参数，其中第一个参数是可执行程序，中间的参数都是可执行程序的参数，最后一个参数必须是NULL
// argv[] 是指针数组，数组中的指针指向的数据和arg 参数是一致的
// envp[] 是指针数组，数组中的每个指针指向一个字符串，该字符串是一个环境变量，例子 "AA=12"
// 如果成功则没有返回值，并且execl()后面的代码不会执行，如果失败则会执行execl()后面的代码，可以使用perror打印错误原因
   
// 进程调用execl()函数族会使用参数传递的程序进行替换堆、栈、代码段、数据段，并没有创建新的进程，原有的进程的PID和进程空间没有变化
~~~
---
 

## 创建子进程，然后由子进程执行其他的程序例子
~~~C
#include <sys/types.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){

    pid_t p;
    int i;
    p=fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p>0){
        printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());
    }

    if(p==0){
        printf("子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
        execl("/bin/ls", "ls", "-alth", NULL);
    }

    return 0;
}
~~~
---
 

# wait()和waitpid()函数
查看方式：man 2 wait
~~~C
#include <sys/types.h>
#include <sys/wait.h>

    pid_t wait(int *status);

    pid_t waitpid(pid_t pid, int *status, int options);
    
// 当子进程先于父进程执行完退出，但是父进程没有调用wait/waitpid回收子进程内核区的PCB资源，则子进程变成僵尸进程
// wait()和waitpid()函数的作用是父进程调用函数会回收子进程的资源和获取子进程的结束状态，注意wait()和waitpid()函数调用一次只能回收一个子进程。

// 注意wait()函数会阻塞当前进程（父进程），等待所有的子进程退出，直到有一个子进程退出就回收这个子进程的资源

// wait函数参数
// status是用于存放子进程结束的状态，后面可以使用宏函数来判断子进程是否正常退出
// WIFEXITED(status)    返回非0表示进程正常结束
// WEXITSTATUS(status)  获取进程的退出状态
// WIFSIGNALED(status)  返回非0表示进程异常终止
// WTERMSIG(status)     获取进程异常终止的信号编号

// wait函数返回值是子进程的pid，子进程正常退出，如果返回-1表示没有这个子进程

// waitpid函数参数
// pid参数分多种情况
// pid=-1时，等待所有的子进程退出，直到一个子进程退出就回收这个子进程的资源
// pid>0时，等待指定pid的子进程退出，回收该子进程的资源
// pid=0时，等待和当前进程在同一个进程组中子进程退出，直到一个子进程退出就回收这个子进程的资源
// pid<-1时，等待进程组ID等于pid绝对值的子进程，直到一个子进程退出就回收这个子进程的资源
// status参数和上述wait函数的status参数用法一致
// options参数是表示waitpid函数是否阻塞当前线程，WNOHANG表示不阻塞，0表示阻塞

// waitpid返回值是子进程的pid，子进程正常退出，如果返回-1表示没有这个子进程，返回值是0（只有options参数是WNOHANG才会出现）表示子进程正在运行
~~~
---
 

## 进程阻塞回收子进程的资源例子，如果需要信号量杀死请使用kill 手动杀死子进程
~~~C
#include <sys/types.h>
#include <sys/wait.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){

    pid_t p;
    int i;
    p=fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p>0){
        int status;
        printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());
        pid_t p = wait(&status);

        printf("终止的子进程pid是%d\n", p);

        if(WIFEXITED(status)){
            printf("子进程正常退出，退出码是%d\n",  WEXITSTATUS(status));
        } else if(WIFSIGNALED(status)){
            printf("子进程被信号量杀死，进程异常终止的信号编号是%d\n", WTERMSIG(status));
        }
    }

    if(p==0){
        printf("子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
        sleep(60);
        return 88;
    }

    return 0;
}

~~~
---
 

## 进程非阻塞回收子进程的资源例子，如果需要信号量杀死请使用kill 手动杀死子进程
~~~C
#include <sys/types.h>
#include <sys/wait.h>
#include <dirent.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){

    pid_t p;
    int i;
    p=fork();
    if(p<0){
        perror("创建子进程失败\n");
        return -1;
    }

    if(p>0){
        int status;
        pid_t wp;
        printf("只有父进程执行的逻辑代码，父进程的pid是%d，父进程的父进程pid是%d\n", getpid(), getppid());

        while(1){
            wp = waitpid(p, &status, WNOHANG);

            if(wp>0){
                printf("终止的子进程pid是%d\n", wp);

                if(WIFEXITED(status)){
                    printf("子进程正常退出，退出码是%d\n",  WEXITSTATUS(status));
                } else if(WIFSIGNALED(status)){
                    printf("子进程被信号量杀死，进程异常终止的信号编号是%d\n", WTERMSIG(status));
                }
            } else if(wp==0){
                printf("子进程正在运行还未退出\n");
            } else if(wp<0){
                printf("没有找到指定的子进程\n");
                break;
            }
        }
    }

    if(p==0){
        printf("子进程执行的逻辑代码，子进程的pid是%d，父进程的pid是%d\n", getpid(), getppid());
        sleep(30);
        return 88;
    }

    return 0;
}

~~~
---
