# 使用TCP协议的聊天室服务端代码如下，使用gcc 编辑时需要添加 -l pthread 参数
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>

// 聊天室中最大成员的数量
#define clients_info_len 100

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define client_str_len 1500


// 全局互斥锁
pthread_mutex_t mutex;

// 客户端的信息
struct client_info{
    // 子线程的ID
    pthread_t thread_id;
    // 客户端的ip地址、端口、协议存储变量
    struct sockaddr_in * client_ip_port;
    // 客户端连接的文件描述符，如果是-1则表示上一个客户端断开了连接，当前整个结构体空间可用
    int cfd;

    // 客户的昵称
    char name[10];
};

// 聊天室中全部的客户端，最多100个客户端
struct client_info clients_info[clients_info_len];

// 根据昵称获取聊天室中客户端连接的文件描述符
int clients_contains(char *name){
    int i;
    for(i=0; i < clients_info_len;i++){
        if(strcmp(clients_info[i].name, name) == 0 ){
            return clients_info[i].cfd;
        }
    }
    return -1;
}

// 获取聊天室中全部已经连接的客户端昵称
void get_client_names(char * client_names){
    int i, j=0;
    for(i=0; i < clients_info_len;i++){
        if(clients_info[i].cfd != -1 && strlen(clients_info[i].name) > 0){
            if(j!=0){
                strcat(client_names, "、");
            }
            strcat(client_names, clients_info[i].name);
            j++;
        }
    }

}

// 检查客户端昵称是否合法
int check_client_name(char * client_name){
    int i;
    for(i=0; i < strlen(client_name);i++){
        if( client_name[i] == '@' || client_name[i] == '$' || client_name[i] == ':' ){
            return -1;
        }
    }
    return 0;
}

// 初始化客户端的信息
void client_info_init(struct client_info *c_info) {
    c_info->client_ip_port = (struct sockaddr_in*) malloc(sizeof(struct sockaddr_in));
    c_info->cfd = -1;
}


void * mythread(void * args){

    struct client_info * c_info = (struct client_info *) args;
    // 获取客户端IP、端口
    char* client_ip = inet_ntoa(c_info->client_ip_port->sin_addr);
    int client_port = ntohs(c_info->client_ip_port->sin_port);
    printf("socket客户端：%s:%d 连接成功\n", client_ip, client_port);

    // write_buf 变量存储发送的数据
    char write_buf[client_str_len];
    // read_buf 变量存储接收的数据
    char read_buf[client_str_len];

    int n=0, i;

    char client_names[1500];

    memset(&client_names, 0, sizeof(client_names));
    strcpy(client_names, "连接服务端成功\n");
    strcat(client_names, "聊天室中的成员：");
    get_client_names(client_names);
    strcat(client_names, "\n请注册，输入昵称");

    // 客户端注册名称
    send(c_info->cfd, client_names, client_str_len, 0);
    while(1){
        // read_buf 变量全部位数归零
        memset(&read_buf, 0, sizeof(read_buf));
        // write_buf 变量全部位数归零
        memset(&write_buf, 0, sizeof(write_buf));

        // 服务端接收的用户昵称
        n = recv(c_info->cfd, &read_buf, sizeof(read_buf), 0);
        if(n == 0){
            printf("socket客户端：%s:%d 已断开连接了\n", client_ip, client_port);
            break;
        }

        printf("服务端读取到客户端 %s:%d 的昵称：%s\n", client_ip, client_port, read_buf);


        if( check_client_name(read_buf)  == -1){
            strcpy(write_buf, "该昵称使用了非法字符'@'或':'，请重新输入昵称");
            send(c_info->cfd, write_buf, client_str_len, 0);
            continue;
        }


        pthread_mutex_lock(&mutex);
        int cfd = clients_contains(read_buf);
        if( cfd != -1 ){
            strcpy(write_buf, "该昵称已经被聊天室中其他人占用，请重新输入昵称");
            send(c_info->cfd, write_buf, client_str_len, 0);
            pthread_mutex_unlock(&mutex);
            continue;
        }

        // 客户端的昵称保存
        if(strlen(c_info->name) == 0){
            strcpy(c_info->name, read_buf);
            // 返回给客户端数据
            strcpy(write_buf, "昵称注册成功");
            send(c_info->cfd, write_buf, client_str_len, 0);
            pthread_mutex_unlock(&mutex);
            break;
        }

        pthread_mutex_unlock(&mutex);
    }

    // 处理客户端发送的消息
    while(1){
        memset(&read_buf, 0, sizeof(read_buf));

        memset(&write_buf, 0, sizeof(write_buf));
        strcpy(write_buf, "你的昵称是：");
        send(c_info->cfd, strcat(write_buf, c_info->name), client_str_len, 0);

        memset(&write_buf, 0, sizeof(write_buf));
        strcpy(write_buf, "聊天室中的成员：");
        get_client_names(write_buf);
        send(c_info->cfd, write_buf, client_str_len, 0);

        memset(&write_buf, 0, sizeof(write_buf));
        strcpy(write_buf, "请按照‘聊天室成员名称:消息’格式进行发送消息，通知聊天室所有人的消息格式是‘@all:消息'，注意空格表示一条消息结束，之后的内容表示给其他人发送消息。接收的消息会按照’消息的发送者$消息‘的格式显示出来。如需下线请输入'exit'");
        send(c_info->cfd, write_buf, client_str_len, 0);

        // 服务端接收的数据
        n = recv(c_info->cfd, &read_buf, sizeof(read_buf), 0);
        if(n == 0){
            printf("socket客户端：%s:%d 已断开连接了\n", client_ip, client_port);
            break;
        }

        printf("服务端读取到客户端 %s:%d 发送的数据：%s\n", client_ip, client_port, read_buf);

        // 用户主动下线
        if( strcmp(read_buf, "exit") == 0 ){
            break;
        }

        char *messages;
        char *target_name;

        target_name = strtok_r(read_buf, ":", &messages);


        if( strlen(messages) == 0 || strlen(target_name) == 0 || check_client_name(target_name) == -1){
            memset(&write_buf, 0, sizeof(write_buf));
            strcpy(write_buf, "聊天格式错误，请重新输入");
            send(c_info->cfd, write_buf, client_str_len, 0);
            continue;
        }

        // 判断是否是发送给所有人的广播消息
        if(strcmp(target_name, "@all") == 0){
            memset(&write_buf, 0, sizeof(write_buf));
            strcat(write_buf, c_info->name);
            strcat(write_buf, "：");
            strcat(write_buf, "@所有人");
            strcat(write_buf, "：");
            strcat(write_buf, messages);
            for(i=0; i < clients_info_len;i++){
                if(clients_info[i].cfd != -1){
                    send(clients_info[i].cfd, write_buf, client_str_len, 0);
                }
            }
            continue;
        }

        // 根据昵称获取接收方连接
        int cfd = clients_contains(target_name);
        if( cfd == -1 ){
            // 返回给客户端数据
            memset(&write_buf, 0, sizeof(write_buf));
            strcat(write_buf, "聊天室中没有找到 ");
            strcat(write_buf, target_name);
            strcat(write_buf, " 用户");
            send(c_info->cfd, write_buf, client_str_len, 0);
        }else{
            memset(&write_buf, 0, sizeof(write_buf));
            strcat(write_buf, c_info->name);
            strcat(write_buf, "$");
            strcat(write_buf, messages);

            printf("返回发送其他客户端的数据：%s\n", write_buf);

            // 返回给其他客户端数据，当然如果其他客户端断开连接就返回聊天室没有该用户客户端
            n = send(cfd, write_buf, client_str_len, 0);
            if(n == 0){
                memset(&write_buf, 0, sizeof(write_buf));
                strcat(write_buf, "聊天室中没有找到 ");
                strcat(write_buf, target_name);
                strcat(write_buf, " 用户");
                send(c_info->cfd, write_buf, client_str_len, 0);
            }
        }

    }
    // 关闭客户端连接的文件描述符
    close(c_info->cfd);
    c_info->cfd = -1;


    // 通知聊天室其他在线的用户，当前用户下线了
    memset(&write_buf, 0, sizeof(write_buf));
    strcat(write_buf, c_info->name);
    strcat(write_buf, " 用户已下线了！");
    for(i=0; i < clients_info_len;i++){
        if(clients_info[i].cfd != -1){
            send(clients_info[i].cfd, write_buf, client_str_len, 0);
        }
    }

    memset(c_info->client_ip_port, 0, sizeof(c_info->client_ip_port));
    memset(c_info->name, 0, sizeof(c_info->name));
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


    // 服务端的socket套接字创建
    int sfd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
    if(sfd == -1){
        perror("socket套接字创建失败\n");
        return -1;
    }

    // 复用ip和port
    int optval=1;
    setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(int));

    // socket套接字绑定服务端的网卡IP、端口
    int re = bind(sfd, (struct sockaddr *)&server_ip_port, sizeof(server_ip_port));
    if(re == -1){
        perror("socket套接字绑定本机ip和端口失败\n");
        return -1;
    }

    // 服务端监听socket套接字
    re = listen(sfd, 1000);
    if(re == -1){
        perror("本机监听socket套接字失败\n");
        return -1;
    }

    // 互斥锁初始化
    pthread_mutex_init(&mutex, NULL);

    // 初始化聊天室
    int i;
    for(i=0; i < clients_info_len;i++){
        struct client_info* args = (struct client_info*) malloc(sizeof(struct client_info));
        client_info_init(args);
        clients_info[i] = *args;
    }


    // 线程属性设置
    pthread_attr_t attr;
    // 初始化线程属性
    pthread_attr_init(&attr);
    // 设置线程属性为分离状态，PTHREAD_CREATE_JOINABLE是非分离属性
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    while(1){
        // 寻找空间存储用户连接信息
        struct client_info* args = NULL;
        for(i=0; i < clients_info_len;i++){
            if(clients_info[i].cfd == -1){
                args = &(clients_info[i]);
                break;
            }
        }

        if(args == NULL){
            sleep(1);
            continue;
        }

        // 获取客户端连接，并且保存客户端连接的IP、端口
        socklen_t c_len = sizeof(*(args->client_ip_port));
        int cfd = accept(sfd, (struct sockaddr *)args->client_ip_port, &c_len);
        if(cfd == -1){
            perror("获取客户端连接失败\n");
            break;
        }

        args->cfd = cfd;

        // 创建线程处理客户端的会话
        re = pthread_create(&args->thread_id, &attr, mythread, args);
        if(re != 0){
            // 线程创建错误通过 strerror 函数查看
            printf("线程创建错误：%s\n", strerror(re));
            return -1;
        }

    }

    // 回收释放线程属性
    pthread_attr_destroy(&attr);

    // 回收聊天室的堆内存
    for(i=0; i < clients_info_len;i++){
        free(clients_info[i].client_ip_port);
        free(&clients_info[i]);
    }

    // 关闭服务端的socket文件描述符
    close(sfd);

    return 0;
}
~~~
---

# 使用TCP协议的聊天室客户端代码如下，使用gcc 编辑时需要添加 -l pthread 参数
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>

// 异步读消息线程回调函数
void * read_messages_thread(void * args){

    int* cfd = (int*) args;
    // read_buf 变量存储接收的数据
    char read_buf[1500];
    int n=0;
    while(1){
        // read_buf 变量全部位数归零
        memset(&read_buf, 0, sizeof(read_buf));
        // 客户端接收的数据
        n = recv(*cfd, &read_buf, sizeof(read_buf), 0);
        if(n == 0){
            printf("socket服务端已断开连接了\n");
            break;
        }
        // 服务端返回的数据
        printf("%s\n", read_buf);
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
        // wirte_buf 变量全部位数归零
        memset(&wirte_buf, 0, sizeof(wirte_buf));

        // 读取客户端用户输入的字符串发送给服务端
        scanf("%s", &wirte_buf);
        n = send(*cfd, &wirte_buf, sizeof(wirte_buf), 0);
        if(n == 0){
            printf("socket服务端已断开连接了\n");
            break;
        }
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
    int cfd = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);

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

    // socket套接字连接服务端
    re = connect(cfd, (struct sockaddr *)&server_ip_port, sizeof(server_ip_port));

    if(re == -1){
        printf("客户端socket套接字连接服务器端失败\n");
        return -1;
    }

    // 线程属性设置
    pthread_attr_t attr;
    // 初始化线程属性
    pthread_attr_init(&attr);
    // 设置线程属性为分离状态，PTHREAD_CREATE_JOINABLE是非分离属性
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

    printf("退出客户端\n");
    return 0;
}
~~~
---