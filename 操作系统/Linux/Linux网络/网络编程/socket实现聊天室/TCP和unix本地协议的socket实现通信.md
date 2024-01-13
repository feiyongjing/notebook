# 使用UDP协议通信的客户端和服务端代码如下

# 服务端代码，使用gcc 编辑时需要添加 -l pthread 参数，运行后当前目录下会多出一个 server.sock 的文件
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <sys/un.h>

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define message_len 1500

void * mythread(void* args){
    int cfd= *(int*)args;
    int i,n;
    while(1){
        char message[message_len];
        n = read(cfd, message, message_len);

        if(n>0){
            printf("接收客户端消息成功：%s\n", message);
        }else if(n==0){
            printf("客户端连接断开了\n");
            break;
        }

        for(i=0;i<n;i++){
            message[i] = toupper(message[i]);
        }

        n = write(cfd, message, n);

        if(n>0){
            printf("返回客户端消息成功：%s\n", message);
        }else if(n<=0){
            printf("返回客户端消息失败：%s\n", message);
        }
    }
    close(cfd);
    free(args);
    pthread_exit(NULL);

}

int main(){

    unlink("./server.sock");
    // 服务端的协议、本地socket文件路径设置
    struct sockaddr_un server_info;
    server_info.sun_family = AF_UNIX;
    strcpy(server_info.sun_path, "./server.sock");

    // 客户端的协议、本地socket文件路径信息
    struct sockaddr_un client_info;
    socklen_t client_info_len = sizeof(client_info);

    // 服务端的socket套接字创建
    int sfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if(sfd == -1){
        perror("socket套接字创建失败\n");
        return -1;
    }

    // socket套接字绑定服务端的本地unix文件
    int re = bind(sfd, (struct sockaddr*)&server_info, sizeof(server_info));
    if(re == -1){
        perror("socket套接字绑定本机socket文件失败\n");
        return -1;
    }

    re = listen(sfd, 1000);
    if(re == -1){
        perror("本机监听socket套接字失败\n");
        return -1;
    }

    pthread_attr_t attr;
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    while(1){
        memset(&client_info, 0, sizeof(client_info));
        int cfd = accept(sfd, (struct sockaddr *)&client_info, &client_info_len);
        if(cfd<0){
            perror("获取客户端连接失败\n");
            continue;
        }
        printf("客户端成功连接到服务端，客户端绑定的socket文件是：%s\n", client_info.sun_path);

        pthread_t thread_id;
        int *pcfd = (int*)malloc(sizeof(int));
        *pcfd = cfd;

        // 创建线程处理客户端的会话
        re = pthread_create(&thread_id, &attr, mythread, (void*)pcfd);
        if(re != 0){
            printf("线程创建错误：%s\n", strerror(re));
            return -1;
        }

    }

    // 回收释放线程属性
    pthread_attr_destroy(&attr);
    // 关闭服务端的socket文件描述符
    close(sfd);
    return 0;
}
~~~
---

# 客户端代码，使用gcc 编辑时需要添加 -l pthread 参数，运行后当前目录下会多出一个 client.sock 的文件
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <sys/un.h>

// 全局互斥锁
pthread_mutex_t mutex;

// 全局条件变量
pthread_cond_t cond;

// 是否发送过消息，而服务端还未返回
int message_is_send =0;


// 异步读消息线程回调函数
void * read_messages_thread(void * args){
    int* cfd = (int*) args;
    // read_buf 变量存储接收的数据
    char read_buf[1500];
    int n=0;
    while(1){
        memset(&read_buf, 0, sizeof(read_buf));

        // 客户端接收的数据
        n = recv(*cfd, &read_buf, sizeof(read_buf), 0);

        if(n==0){
            printf("\n服务端的断开连接了\n");
            break;
        }
        pthread_mutex_lock(&mutex);
        printf("服务端返回的数据：%s\n", read_buf);
        pthread_mutex_unlock(&mutex);
        message_is_send = 0;
        pthread_cond_signal(&cond);
    }
    close(*cfd);
    *cfd = -1;
    // 退出当前线程
    pthread_exit(NULL);
}

// 异步写消息线程回调函数
void * write_messages_thread(void * args){
    int* cfd = (int*) args;

    // wirte_buf 变量存储发送的数据
    char wirte_buf[100] = "hello world\n";
    int n=0;
    while(1){
        memset(&wirte_buf, 0, sizeof(wirte_buf));

        pthread_mutex_lock(&mutex);
        if(message_is_send>0){
            pthread_cond_wait(&cond, &mutex);
        }
        message_is_send=1;
        // 读取客户端用户输入的字符串发送给服务端
        printf("请输入需要转换为大写的字符串：");
        scanf("%s", &wirte_buf);
        pthread_mutex_unlock(&mutex);
        n = send(*cfd, &wirte_buf, sizeof(wirte_buf), 0);
        if(n==0){
            printf("\n服务端的断开连接了\n");
            break;
        }
    }
    close(*cfd);
    *cfd = -1;
    // 退出当前线程
    pthread_exit(NULL);
}

int main(){
    unlink("./client.sock");
    // 客户端的协议、本地socket文件路径设置
    struct sockaddr_un client_info;
    client_info.sun_family = AF_UNIX;
    strcpy(client_info.sun_path, "./client.sock");

    // 服务端的协议、本地socket文件路径信息
    struct sockaddr_un server_info;
    server_info.sun_family = AF_UNIX;
    strcpy(server_info.sun_path, "./server.sock");

    // 客户端的socket套接字创建
    int cfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if(cfd == -1){
        printf("socket套接字创建失败\n");
        return -1;
    }

    // socket套接字绑定本地服务端socket文件
    int re = bind(cfd, (struct sockaddr *)&client_info, sizeof(client_info));
    if(re == -1){
        printf("socket套接字绑定本机服务端socket文件失败：%s\n",strerror(re));
        return -1;
    }

    // socket套接字连接服务端
    re = connect(cfd, (struct sockaddr *)&server_info, sizeof(server_info));
    if(re == -1){
        printf("客户端socket套接字连接服务器端失败\n");
        return -1;
    }

    // 互斥锁初始化
    pthread_mutex_init(&mutex, NULL);
    // 条件变量初始化
    pthread_cond_init(&cond, NULL);

    // 线程属性设置
    pthread_attr_t attr;
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    pthread_t read_thread_id;
    pthread_t write_thread_id;

    int re1 = pthread_create(&read_thread_id, &attr, read_messages_thread, &cfd);
    int re2 = pthread_create(&write_thread_id, &attr, write_messages_thread, &cfd);

    if(re1 != 0){
        printf("线程创建错误：%s\n", strerror(re1));
        return -1;
    }
    if(re2 != 0){
        printf("线程创建错误：%s\n", strerror(re2));
        return -1;
    }

    // 阻塞主线程直到连接断开
    while(1){
        sleep(1);
        if(cfd==-1){
            break;
        }
    }

    // 回收释放线程属性
    pthread_attr_destroy(&attr);

    // 回收互斥锁
    pthread_mutex_destroy(&mutex);
    // 回收条件变量
    pthread_cond_destroy(&cond);
    return 0;
}
~~~


