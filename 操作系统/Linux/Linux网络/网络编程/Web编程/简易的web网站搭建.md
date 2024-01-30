# 使用了socket、epoll实现网站搭建，代码如下
### 注意如果客户端和服务器端在不同的机器上运行需要修改服务端socket绑定的ip地址和客户端中连接服务端的ip地址
### 服务端的socket绑定的IP地址绝对不能使用127.0.0.1，否则只有服务端所在机器可以运行客户端进行连接
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <sys/stat.h>
#include <fcntl.h>

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define message_len 20000

// 监听的事件数量
#define event_len 1000

typedef struct socket_events_data{
    int fd;
    int events;
    void (*call_back)(int sfd, int events, void* args);
} socket_events_data;


// 服务端的ip地址(注意如果使用127.0.0.1就只能当前服务器可以连接到socket)、端口、服务端最大连接数
int tcp4bind(short port, const char *ip, int num){

    struct sockaddr_in server_ip_port;
    memset(&server_ip_port, 0, sizeof(server_ip_port));

    server_ip_port.sin_family = AF_INET;
    server_ip_port.sin_port = htons(port);

    if(ip == NULL){
        // 使用本机随机的网口，INADDR_ANY表示0.0.0.0
        server_ip_port.sin_addr.s_addr = htonl(INADDR_ANY);
    }else{
        inet_pton(AF_INET, ip, &server_ip_port.sin_addr.s_addr);
    }

    // 服务端的socket套接字创建
    int sfd = socket(AF_INET, SOCK_STREAM, 0);
    if(sfd == -1){
        perror("socket套接字创建失败\n");
        exit(1);
    }

    // 复用ip和port
    int optval=1;
    setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(int));

    // socket套接字绑定服务端的网卡IP、端口
    int re = bind(sfd, (struct sockaddr*)&server_ip_port, sizeof(server_ip_port));
    if(re == -1){
        perror("socket套接字绑定本机ip和端口失败\n");
        exit(1);
    }

    // 服务端监听socket套接字
    re = listen(sfd, num);
    if(re == -1){
        perror("本机监听socket套接字失败\n");
        exit(1);
    }
    return sfd;
}

ssize_t read_line(int fd, char *buffer, size_t size) {
    ssize_t i = 0;
    char c;
    while (i < size - 1) {
        ssize_t res = read(fd, &c, 1);
        if (res == -1) {
            printf("读取文件描述符一行错误\n");
            return -1;
        } else if (res <= 0) {
            // 读到文件末尾了
            break;
        }
        buffer[i++] = c;
        if (c == '\n') {
            break;
        }
    }
    buffer[i] = '\0';
    return i;
}

const char* get_content_type(const char* filename) {
    const char* ext = strrchr(filename, '.');
    if (strcmp(ext, ".html") == 0 || strcmp(ext, ".htm") == 0) {
        return "text/html";
    } else if (strcmp(ext, ".css") == 0) {
        return "text/css";
    } else if (strcmp(ext, ".js") == 0) {
        return "application/javascript";
    } else if (strcmp(ext, ".jpg") == 0 || strcmp(ext, ".jpeg") == 0) {
        return "image/jpeg";
    } else if (strcmp(ext, ".png") == 0) {
        return "image/png";
    } else if (strcmp(ext, ".gif") == 0) {
        return "image/gif";
    } else if (strcmp(ext, ".txt") == 0) {
        return "text/plain; charset=utf-8";
    } else {
        return "application/octet-stream";
    }
}

int send_header(int cfd, const char* code, const char* msg, const char* ext, int body_len){
    char buf[message_len] = {0};
    sprintf(buf, "HTTP/1.1 %s %s\r\n", code, msg);
    sprintf(buf + strlen(buf), "Content-Type: %s\r\n", ext);
    if(body_len>0){
        sprintf(buf + strlen(buf), "Content-Length: %d\r\n", body_len);
    }
    sprintf(buf + strlen(buf), "\r\n");

    write(cfd, buf, strlen(buf));
    return 0;
}



int send_file(int cfd, const char* filename){
    int fd = open(filename, O_RDONLY);
    if(fd<0){
        perror("该文件打开失败");
        return -1;
    }

    char buf[message_len] = {0};
    int n;

    while(1){
        memset(buf, 0, message_len);
        n=read(fd, buf, message_len);
        if(n > 0){
            write(cfd, buf, n);
            printf("写响应体：\n%s\n", buf);
            printf("响应体大小：\n%d\n", n);
        }else{
            break;
        }
    }

    close(fd);
    return 0;
}


int http_request(int cfd, int epfd){
    int n;
    char buf[message_len];

    memset(buf, 0, message_len);

    n = read_line(cfd, buf, message_len);

    if(n<=0){
        close(cfd);
        epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, NULL);
        return 0;
    }

    char reqType[16] = {0};
    char fileName[255] = {0};
    char protocal[16] = {0};
    sscanf(buf, "%[^ ] /%[^ ] %[^ \r\n]", reqType, fileName, protocal);

    printf("请求方式是：%s\n", reqType);
    printf("请求路径是：%s\n", fileName);
    printf("请求协议版本是：%s\n", protocal);

    while ((n=read_line(cfd, buf, message_len)) > 0){
        // 如果是阻塞读取会导致永远阻塞，需要退出循环读取
        //if (n == 2 && buf[0] == '\r' && buf[1] == '\n') {
        //    break;
        //}
        printf("%s", buf);
    }

    struct stat st;
    if(stat(fileName, &st) <0){
        printf("该路径的文件不存在\n");
        send_header(cfd, "404", "Not Found", get_content_type("error.html"), 0);
        send_file(cfd, "error.html");
    }else{
        printf("返回该路径的文件: %s\n", fileName);
        printf("文件类型是: %s\n", get_content_type(fileName));
        printf("文件大小是: %d\n", st.st_size);
        send_header(cfd, "200", "OK", get_content_type(fileName), st.st_size);
        printf("准备返回响应体\n");
        send_file(cfd, fileName);
    }

    return 0;
}

int main(){
    // 改变工作目录，防止工作目录所在的设备被物理拔出导致进程退出
    chdir("/root/test");

    int sfd = tcp4bind(9311, "0.0.0.0", 1000);

    int epfd = epoll_create(1024);
    if(epfd < 0){
        perror("epoll树创建失败\n");
        exit(1);
    }

    // socket_events_data sfd_events_data;
    // memset(&sfd_events_data, 0, sizeof(sfd_events_data));

    struct epoll_event ev;
    ev.events=EPOLLIN;
    ev.data.fd = sfd;

    epoll_ctl(epfd, EPOLL_CTL_ADD, sfd, &ev);

    int nready, i, fd, cfd, n, flags;
    char message[message_len];

    struct sockaddr_in client_ip_port;
    socklen_t c_len = sizeof(client_ip_port);

    struct epoll_event sfd_events[event_len];
    while(1){
        nready = epoll_wait(epfd, sfd_events, event_len, -1);
        if(nready <0 ){
            continue;
        }
        for(i=0;i < nready;i++){
            fd = sfd_events[i].data.fd;
            // 获取监听的事件，判断是客户端连接事件，还是客户端发送数据事件
            if(fd == sfd){
                cfd = accept(sfd, NULL, NULL);
                if(cfd == -1){
                    perror("获取客户端连接失败\n");
                    continue;
                }

                flags = fcntl(cfd, F_GETFL, 0);
                fcntl(cfd, F_SETFL, flags | O_NONBLOCK);

                printf("客户端连接上树\n");
                ev.events = EPOLLIN | EPOLLET;
                ev.data.fd = cfd;
                epoll_ctl(epfd, EPOLL_CTL_ADD, cfd, &ev);
            }else{
                printf("客户端接收到消息-------------------------------\n");
                http_request(fd, epfd);
            }
        }
    }

    return 0;
}
~~~