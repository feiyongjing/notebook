# 使用UDP协议通信的客户端和服务端代码如下

# 服务端代码
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define message_len 1500

int main(){
    // 服务端的ip地址、端口、协议设置
    struct sockaddr_in server_ip_port;
    memset(&server_ip_port, 0, sizeof(server_ip_port));

    server_ip_port.sin_family = AF_INET;
    server_ip_port.sin_port = htons(9311);
    inet_pton(AF_INET, "127.0.0.1", &server_ip_port.sin_addr.s_addr);
    // server_ip_port.sin_addr.s_addr = htonl(INADDR_ANY); INADDR_ANY 表示本机可用随机的网卡

    // 服务端的socket套接字创建
    int sfd = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if(sfd == -1){
        perror("socket套接字创建失败\n");
        return -1;
    }

    // 复用ip和port
    int optval=1;
    setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(int));

    // socket套接字绑定服务端的网卡IP、端口
    int re = bind(sfd, (struct sockaddr*)&server_ip_port, sizeof(server_ip_port));
    if(re == -1){
        perror("socket套接字绑定本机ip和端口失败\n");
        return -1;
    }

    // 客户端的ip地址、端口、协议设置
    struct sockaddr_in client_ip_port;
    int client_ip_port_len = sizeof(client_ip_port);

    int i,n;

    while(1){
        memset(&client_ip_port, 0, sizeof(client_ip_port));

        char message[message_len];
        n = recvfrom(sfd, message, message_len, 0, (struct sockaddr*)&client_ip_port, &client_ip_port_len);


        // 获取客户端IP、端口
        char* client_ip = inet_ntoa(client_ip_port.sin_addr);
        int client_port = ntohs(client_ip_port.sin_port);


        if(n>0){
            printf("接收客户端：%s:%d消息成功：%s\n", client_ip, client_port, message);
        }

        for(i=0;i<n;i++){
            message[i] = toupper(message[i]);
        }

        n = sendto(sfd, message, n, 0, (struct sockaddr*)&client_ip_port, sizeof(client_ip_port));

        if(n>0){
            printf("返回客户端：%s:%d消息成功：%s\n", client_ip, client_port, message);
        }else if(n<=0){
            printf("返回客户端：%s:%d消息失败：%s\n", client_ip, client_port, message);
        }
    }

    // 关闭服务端的socket文件描述符
    close(sfd);

    return 0;
}
~~~
---

# 客户端代码，使用gcc 编辑时需要添加 -l pthread 参数
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>

// 全局互斥锁
pthread_mutex_t mutex;

// 全局条件变量
pthread_cond_t cond;

// 是否发送过消息，而服务端还未返回
int message_is_send =0;


struct server_info{
    // 服务端的ip地址、端口、协议存储变量
    struct sockaddr_in * server_ip_port;
    // 服务端连接的文件描述符
    int cfd;
};

// 异步读消息线程回调函数
void * read_messages_thread(void * args){

    struct sockaddr_in server_ip_port;
    int server_ip_port_len = sizeof(server_ip_port);
    int* cfd = (int*) args;
    // read_buf 变量存储接收的数据
    char read_buf[1500];
    int n=0;
    while(1){
        memset(&read_buf, 0, sizeof(read_buf));

        // 客户端接收的数据
        n = recvfrom(*cfd, &read_buf, sizeof(read_buf), 0, (struct sockaddr*)&server_ip_port, &server_ip_port_len);

        // 获取服务端IP、端口
        char* server_ip = inet_ntoa(server_ip_port.sin_addr);
        int server_port = ntohs(server_ip_port.sin_port);

        if(n<=0){
            printf("接收服务端：%s:%d的数据出错了", server_ip, server_port);
        }
        pthread_mutex_lock(&mutex);
        printf("服务端：%s:%d返回的数据：%s\n", server_ip, server_port, read_buf);
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

    struct server_info* s_info = (struct server_info*) args;
    struct sockaddr * server_ip_port = (struct sockaddr*) s_info->server_ip_port;
    int* cfd = &s_info->cfd;
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
        n = sendto(*cfd, &wirte_buf, sizeof(wirte_buf), 0, server_ip_port, sizeof(*server_ip_port));
    }
    close(*cfd);
    *cfd = -1;
    // 退出当前线程
    pthread_exit(NULL);
}

int main(){
    // 服务端的ip地址、端口、协议设置
    struct sockaddr_in server_ip_port;
    memset(&server_ip_port, 0, sizeof(server_ip_port));

    server_ip_port.sin_family = AF_INET;
    server_ip_port.sin_port = htons(9311);
    inet_pton(AF_INET, "127.0.0.1", &server_ip_port.sin_addr.s_addr);
    // server_ip_port.sin_addr.s_addr = htonl(INADDR_ANY); INADDR_ANY 表示本机可用随机的网卡

    // 客户端的ip地址、端口、协议设置
    struct sockaddr_in client_ip_port;
    memset(&client_ip_port, 0, sizeof(client_ip_port));

    client_ip_port.sin_family = AF_INET;
    //client_ip_port.sin_port = htons(9312); 不设置客户端的端口，客户端启动时会随机使用端口
    client_ip_port.sin_addr.s_addr = htonl(INADDR_ANY);

    // 客户端的socket套接字创建
    int cfd = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);

    if(cfd == -1){
        printf("socket套接字创建失败\n");
        return -1;
    }

    // socket套接字绑定本地ip地址和端口
    int re = bind(cfd, (struct sockaddr *)&client_ip_port, sizeof(client_ip_port));

    if(re == -1){
        printf("socket套接字绑定本机ip和端口失败\n");
        return -1;
    }

    struct server_info s_info;
    s_info.server_ip_port = &server_ip_port;
    s_info.cfd = cfd;

    // 互斥锁初始化
    pthread_mutex_init(&mutex, NULL);
    // 条件变量初始化
    pthread_cond_init(&cond, NULL);

    // 线程属性设置
    pthread_attr_t attr;
    // 初始化线程属性
    pthread_attr_init(&attr);
    // 设置线程属性为分离状态，PTHREAD_CREATE_JOINABLE是非分离属性
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    pthread_t read_thread_id;
    pthread_t write_thread_id;

    int re1 = pthread_create(&read_thread_id, &attr, read_messages_thread, &cfd);
    int re2 = pthread_create(&write_thread_id, &attr, write_messages_thread, &s_info);

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

    printf("退出客户端\n");
    return 0;
}
~~~