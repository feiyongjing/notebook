## libevent介绍    
### 参考 https://blog.csdn.net/Lemon_tea666/article/details/92637297
#### Libevent 是一个用C语言编写的、轻量级的开源高性能事件通知库，主要有以下几个亮点：事件驱动（ event-driven），高性能;轻量级，专注于网络，不如 ACE 那么臃肿庞大；源代码相当精炼、易读；跨平台，支持 Windows、 Linux、 *BSD 和 Mac Os；支持多种 I/O 多路复用技术， epoll、 poll、 dev/poll、 select 和 kqueue 等；支持 I/O，定时器和信号等事件；注册事件优先级。
#### Libevent 已经被广泛的应用，作为底层的网络库；比如 memcached、 Vomit、 Nylon、 Netchat等等。  
---

# libevent特点和组成

### libevent的特点和优势
事件驱动，高性能；
轻量级，专注于网络；
跨平台，支持 Windows、Linux、Mac Os等；
支持多种 I/O多路复用技术， epoll、poll、dev/poll、select 和kqueue 等；
支持 I/O，定时器和信号等事件；

### libevent的组成
事件管理包括各种IO（socket）、定时器、信号等事件，也是libevent应用最广的模块；
缓存管理是指evbuffer功能；
DNS是libevent提供的一个异步DNS查询功能；
HTTP是libevent的一个轻量级http实现，包括服务器和客户端
---

# 安装libevent 
1. 从 http://libevent.org 下载源码包，目前我安装的是 libevent-2.1.12-stable.tar.gz
2. 解压源码包，tar -zxvf libevent-2.1.12-stable.tar.gz
3. 进入 libevent-2.1.12-stable 目录查看README文件查看详细的安装步骤，我的安装步骤如下
4. 进入源码目录，执行 ./configure 检测安装环境生成 makefile，注意在安装时可以添加参数指定libevent的安装路径，例如 ./configure --prefix=/usr/xxx ，但是在进行源码包执行 make 编译时需要添加 -l 参数指定源码编译所需的头文件和添加 -L 参数指定源码编译所需的库文件。若默认安装时不指定 --prefix 参数则会安装在系统默认的路径下，编译时无需添加参数指定头文件和库文件，默认会将头文件安装在 /usr/local/include 目录 而库文件安装在 /usr/local/lib 目录下
5. 执行 make 命令编译
6. 执行 sudo make install 命令进行安装
7. 检测是否安装 ls -al /usr/local/lib | grep libevent
8. 添加环境变量 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/ 注意只对当前的shell起作用，每次使用时仍需设置，如需永久生效请修改 /etc/ld.so.conf 配置文件 sudo echo "/usr/local/lib" >> /etc/ld.so.conf 然后使配置生效
7. 在自己写的程序中如果使用了libevent库，则需要在编译时添加 -l event 进行链接编译（如果不是libevent库默认安装在系统目录下，还需要使用 -l 和 -L 参数添加libevent库安装路径下的头文件和库文件），例如源码的 sample 目录中有例子hello-world.c 程序，进行编译程序 gcc -o hello-world.o hello-world.c -l event ， 运行程序 ./hello-world.o ，再开一个终端运行 nc 127.1 9995 会打印 Hello, World!

### 2.0以上版本的API及调用顺序
~~~c
#include <event2/event.h>

    // event_base_new 函数创建event_base结构体，在不同的操作系统中event_base调用不同的多路IO接口去判断事件是否激活，在linux中是使用的epoll，当然也可以切换
    // event_base结构体持有事件集合，event_base相当于epoll树的根节点，可以监测事件是否被集合从而执行事件回调方法
    // 成功创建返回event_base结构体指针，失败返回NULL
    struct event_base* event_base_new(void);

    // 某些事件不能跨进程存活。在fork出子进程后，子进程需要使用event_reinit()函数重新初始化event_base结构体
    // base 参数是event_base
    // 如果成功，返回0，如果某些事件不能重新添加，返回-1
    int event_reinit(struct event_base *base);

    // event_new 函数创建事件
    // base 参数是event_base
    // fd 参数是监控的文件描述符
    // events 参数是需要监控的事件类型和设置，EV_TIMEOUT 是超时事件、EV_READ是读事件、EV_WRITE是写事件、EV_SIGNAL是信号事件、EV_PERSIST表示会周期性监控事件、EV_ET是边缘触发事件（只有event_base调用的多路IO接口支持ET（例如 epoll），才会生效），上述事件可以组合的添加，例如周期性的监控读时间使用 EV_READ | EV_PERSIST
    // cb 参数是事件触发后的回调函数
    // arg 参数是调用回调函数时传递的参数
    // 成功返回事件指针
    struct event *event_new(struct event_base* base, evutil_socket_t fd, short events, event_callback_fn cb, void* arg);
    // 自定义事件回调函数接口，自定义的函数需要与如下的my_event 函数参数返回值一致
    typedef void (*my_event)(evutil_socket_t fd, short events, void* arg);

    // event_add 函数添加事件到event_base进行监控事件
    // ev 参数是事件
    // timeout 参数设置超时时间，结构体如下
    // 如果成功，返回0，如果某些事件不能重新添加，返回-1
    int event_add(struct event *ev, const struct timeval *timeout);

    // event_del 函数从event_base下线监控的事件
    // ev 参数是需要下线事件
    // 如果成功，返回0，失败返回-1
    int event_del(struct event * ev);

    // event_base_dispatch事件循环，会一直监听事件，直到没有事件被监听或者是被结束循环的API终止
    // base 参数是event_base
    // 如果成功，返回0，如果某些事件不能重新添加，返回-1
    int event_base_dispatch(struct event_base *base);

    // event_base_lookexit和event_base_lookbreak会终止事件循环
    // event_base_lookexit是等待timeout参数超时时间后终止，如果期间有事件回调或者正在执行事件回调，则在事件回调结束后终止，event_base_lookbreak是立刻终止
    // base 参数是event_base
    // timeout 参数设置超时时间，结构体如下
    // 如果成功，返回0，失败返回-1
    int event_base_lookexit(struct event_base *base, const struct timeval *timeout);
    int event_base_lookbreak(struct event_base *base);

    struct timeval {
        time_t      tv_sec;         /* 秒 */
        suseconds_t tv_usec;        /* 微秒 */
    };


    // event_free 函数释放事件
    // ev 参数是需要释放的事件
    void event_free(struct event* ev);

    // event_base_free释放event_base
    // base 参数是需要释放的event_base
    void event_base_free(struct event_base* base);

~~~

# 以下是使用 libevent 监听socket连接通信

### 服务端代码，注意需要添加 -l event -l pthread 参数进行编译，并且需要提前安装好 libevent 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <event2/event.h>
#include <pthread.h>

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define message_len 1500

void* my_thread(void* arg){
    int cfd= *(int*)arg;
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
            break;
        }
    }
    close(cfd);
    free(arg);
    pthread_exit(NULL);
}

// 监控客户端socket连接
void conncb(evutil_socket_t fd, short events, void* arg){
    pthread_attr_t* attr = (pthread_attr_t*) arg;
    // 客户端的ip、port
    struct sockaddr_in client_ip_port;
    socklen_t c_len = sizeof(client_ip_port);
    int cfd = accept(fd, (struct sockaddr *)&client_ip_port, &c_len);
    if(cfd<0){
        perror("获取客户端连接失败\n");
        return;
    }

    pthread_t thread_id;
    int *pcfd = (int*)malloc(sizeof(int));
    *pcfd = cfd;

    //创建线程处理客户端的会话
    int re = pthread_create(&thread_id, attr, my_thread, (void*)pcfd);
    if(re != 0){
        printf("线程创建错误：%s\n", strerror(re));
    }
    return;
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
    int sfd = socket(AF_INET, SOCK_STREAM, 0);
    if(sfd == -1){
        perror("socket套接字创建失败\n");
        return -1;
    }

    // socket套接字绑定ip、port
    int re = bind(sfd, (struct sockaddr*)&server_ip_port, sizeof(server_ip_port));
    if(re == -1){
        perror("socket套接字绑定ip、port失败\n");
        return -1;
    }

    re = listen(sfd, 1000);
    if(re == -1){
        perror("监听socket套接字失败\n");
        return -1;
    }

    // 线程属性设置
    pthread_attr_t attr;
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

    struct event_base* base = event_base_new();

    struct event* ev = event_new(base, sfd, EV_READ | EV_PERSIST, conncb, &attr);
    event_add(ev, NULL);

    // 开始事件循环监控
    event_base_dispatch(base);

    // 回收释放线程属性
    pthread_attr_destroy(&attr);

    event_free(ev);
    event_base_free(base);

    close(sfd);
    return 0;
}
~~~
---

### 客户端代码，注意需要添加 -l pthread 参数进行编译
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
int message_is_send = 0;


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
    // client_ip_port.sin_port = htons(9312); 不设置客户端的端口，客户端启动时会随机使用端口
    client_ip_port.sin_addr.s_addr = htonl(INADDR_ANY);

    // 客户端的socket套接字创建
    int cfd = socket(AF_INET, SOCK_STREAM, 0);
    if(cfd == -1){
        printf("socket套接字创建失败\n");
        return -1;
    }

    // socket套接字绑定本地服务端socket文件
    int re = bind(cfd, (struct sockaddr *)&client_ip_port, sizeof(client_ip_port));
    if(re == -1){
        printf("socket套接字绑定失败：%s\n",strerror(re));
        return -1;
    }

    // socket套接字连接服务端
    re = connect(cfd, (struct sockaddr *)&server_ip_port, sizeof(server_ip_port));
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
---

# bufferevent 的API使用
~~~c
#include <event2/listener.h>
#include <event2/bufferevent.h>

    // bufferevent_socket_new 函数创建一个 struct bufferevent ，本质还是一个 event
    // base 参数是event_base
    // fd 参数是监控的文件描述符
    // 参数options: 例如：BEV_OPT_CLOSE_ON_FREE 释放 bufferevent 时关闭底层传输端口
    // 成功时返回bufferevent,失败则返回NULL
    struct bufferevent * bufferevent_socket_new(struct event_base *base, evutil_socket_t fd, enum bufferevent_options options);

    // bufferevent_free 函数释放bufferevent
    // bev 参数是需要释放的bufferevent
    void bufferevent_free(struct bufferevent *bev);

    //对bufferevent设置回调函数	
    // readcb参数设置读回调函数（参考下面的模板），读回调是当从监控文件描述符的读缓冲区读取数据到 bufferevent读缓冲区 时的回调函数
    // writecb设置写回调函数（参考下面的模板），写回调是当从bufferevent写缓冲区写数据到 监控文件描述符的写缓冲区 时的回调函数
    // eventcb设置事件回调函数（参考下面的模板）
	void bufferevent_setcb(
			struct bufferevent *bufev,
			bufferevent_data_cb readcb,    
			bufferevent_data_cb writecb,   	      
			bufferevent_event_cb eventcb,  	
			void *cbarg
	);

    // 读写回调函数模板
	typedef void (*bufferevent_data_cb)(struct bufferevent *bev, void *ctx);
    // 事件回调函数模板
    // events参数:
    //	BEV_EVENT_READING： 读取操作时发生某事件，具体是哪种事件请看其他标志
    //	BEV_EVENT_WRITING：写入操作时发生某事件，具体是哪种事件请看其他标志
    //	BEV_EVENT_ERROR：  操作时发生错误
    //	BEV_EVENT_TIMEOUT：发生超时
    //	BEV_EVENT_EOF：    遇到文件结束指示
    //	BEV_EVENT_CONNECTED：请求的连接过程已经完成实现客户端的时候可以判断
	typedef void (*bufferevent_event_cb)(struct bufferevent *bev, short events, void *ctx);

    // 开启 bufferevent 事件指定的回调
    // bev 参数是bufferevent
    // events 参数设置需要开启监控的事件类型和设置
	void bufferevent_enable(struct bufferevent *bufev, short events); 

    // 禁用 bufferevent 事件指定的回调
    // bev 参数是bufferevent
    // events 参数设置禁用的监控的事件类型和设置
	void bufferevent_disable(struct bufferevent *bufev, short events); 

    // 查看 bufferevent 事件开启监控的事件类型和设置
    // bev 参数是bufferevent
    // 返回开启监控的事件类型和设置
	short bufferevent_get_enabled(struct bufferevent *bufev)

    // 向bufferevent的写缓冲区写入数据，当然bufferevent的写缓冲区数据满了内核会从bufferevent写缓冲区写数据到 监控文件描述符的写缓冲区 并且触发 bufferevent 的写回调函数
    // bev 参数是bufferevent
    // data 参数是写入的数据，size 参数是写入数据的字节数
    // 成功返回0，失败返回-1
    int bufferevent_write(struct bufferevent *bufev, const void *data, size_t size);

    // 从bufferevent的输入缓冲区读取数据，当然内核会循环的从监控文件描述符的读缓冲区读取数据到 bufferevent读缓冲区 并且触发 bufferevent 的读回调函数
    // bev 参数是bufferevent
    // data 参数是读取的数据存储地址，size 参数是读取数据的字节数
    // 成功返回读取的字节数
    size_t bufferevent_read(struct bufferevent *bufev, void *data, size_t size);

    // bufferevent_socket_connect 函数用于客户端与服务端建立socket连接，实际就是 socket、bind、connect 函数的封装
    // bev 参数是bufferevent
    // addr 参数设置服务端的 ip 和 port
    // addrlen 参数设置 addr 参数占用的内存大小
    // 成功返回零，失败返回-1，并且设置errno
    int bufferevent_socket_connect(struct bufferevent *bev, struct sockaddr *address, int addrlen);

    // evconnlistener_new_bind 函数生成链接监听器，实际就是 socket、bind、listen、accept 函数的封装
    // base 参数是event_base
    // evconnlistener_cb 参数是当有新的socket连接后的回调函数（调用socket的 accept 函数之后调用的回调函数），参考下面的回调函数模板
    // ptr 参数是 evconnlistener_cb 回调函数的参数
    // flags 参数设置为 LEV_OPT_LEAVE_SOCKETS_BLOCKING 则表示阻塞，LEV_OPT_CLOSE_ON_FREE 表示关闭时自动释放内部socket连接的文件描述符，LEV_OPT_REUSEABLE 表示端口复用，LEV_OPT_THREADSAFE 表示分配锁（线程安全）
    // backlog 参数是socket同时通信的最大连接数，如果是-1则会自动设置一个合适的值，是0则内部不会调用 socket的listen 函数
    // sa 参数是设置服务端的 ip 和 port
    // socklen 参数是 sa 参数占用的内存大小
    // 成功返回生成链接监听器
    struct evconnlistener *evconnlistener_new_bind(
                struct event_base *base, evconnlistener_cb cb, void *ptr,
                unsigned flags,
                int backlog,
                const struct sockaddr *sa, 
                int socklen
    );

    // 当有新的socket连接后的回调函数模板
    // listener 参数是链接监听器
    // sock 参数是用于socket通信的文件描述符
    // addr 参数设置客户端的 ip 和 port
    // len 参数设置 addr 参数占用的内存大小
    // ptr 参数是自定义回调函数的参数
    typedef void (*evconnlistener_cb)( 
                struct evconnlistener *listener, evutil_socket_t sock,
                struct sockaddr *addr,
                int len,
                void *ptr
    );

    // 释放链接监听器
    void evconnlistener_free(struct evconnlistener *lev);

    // 使链接监听器失效
    int evconnlistener_disable(struct evconnlistener *lev);

    // 使链接监听器生效
    int evconnlistener_enable(struct evconnlistener *lev);

    // 设置链接监听器的回调函数，可以用于重新设置链接监听器的回调函数
    // lev 参数是链接监听器
    // evconnlistener_cb 参数是当有新的socket连接后的回调函数（调用socket的 accept 函数之后调用的回调函数），参考下面的回调函数模板
    // arg 参数是 evconnlistener_cb 回调函数的参数
    void evconnlistener_set_cb(struct evconnlistener *lev, evconnlistener_cb cb, void *arg);
~~~
---

## 以下是使用 libevent 的bufferevent API 监听socket连接通信

### 服务端代码，注意需要添加 -l event 参数进行编译，并且需要提前安装好 libevent 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <event2/event.h>
#include <event2/listener.h>
#include <event2/bufferevent.h>

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define message_len 1500

// bufferevent 缓冲区的读回调函数
void bufferevent_read_data_cb(struct bufferevent *bev, void *ctx){
    int i,n;
    char message[message_len];
    n = bufferevent_read(bev, message, message_len);

    if(n>0){
        printf("接收客户端消息成功：%s\n", message);
        for(i=0;i<n;i++){
            message[i] = toupper(message[i]);
        }
        bufferevent_write(bev, message, n);
        printf("返回客户端消息成功：%s\n", message);
    }else if(n<0){
    }
}

// bufferevent 缓冲区的事件回调
void event_cb(struct bufferevent *bev, short events, void *arg){
    if(events & BEV_EVENT_ERROR){
        printf("some other error\n");
        bufferevent_free(bev);
    }
}

// 创建 bufferevent，并设置它的缓冲区读写回调函数
void my_evlistener_cb(
        struct evconnlistener *listener, evutil_socket_t sock,
        struct sockaddr *addr, int len, void *ptr){
    struct event_base* base = (struct event_base*) ptr;
    struct bufferevent* buffer_ev = bufferevent_socket_new(base, sock, BEV_OPT_CLOSE_ON_FREE);

    bufferevent_setcb(buffer_ev, bufferevent_read_data_cb, NULL, event_cb, NULL);

    bufferevent_enable(buffer_ev, EV_READ | EV_PERSIST);
    bufferevent_enable(buffer_ev, EV_WRITE);
    //bufferevent_enable(buffer_ev, EV_ERROR | EV_EOF);
}

int main(){
    // 服务端的ip地址、端口、协议设置
    struct sockaddr_in server_ip_port;
    memset(&server_ip_port, 0, sizeof(server_ip_port));

    server_ip_port.sin_family = AF_INET;
    server_ip_port.sin_port = htons(9311);
    inet_pton(AF_INET, "127.0.0.1", &server_ip_port.sin_addr.s_addr);

    struct event_base* base = event_base_new();

    // 服务端的socket套接字创建
    struct evconnlistener* evlistener = evconnlistener_new_bind(
            base, my_evlistener_cb, base,
            LEV_OPT_CLOSE_ON_FREE | LEV_OPT_REUSEABLE,
            1000,
            (struct sockaddr*)&server_ip_port,
            sizeof(server_ip_port)
            );

    // 开始事件循环监控
    event_base_dispatch(base);

    evconnlistener_free(evlistener);
    event_base_free(base);
    return 0;
}
~~~

### 客户端代码，注意需要添加 -l event 参数进行编译，并且需要提前安装好 libevent 库
~~~c
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <event2/event.h>
#include <event2/bufferevent.h>

// socket套接字每次接收和发送的数据长度固定，防止粘包
#define message_len 1500

// bufferevent 缓冲区的读回调函数
void bufferevent_read_data_cb(struct bufferevent *bev, void *ctx){
    char message[message_len];
    // 客户端接收的数据
    bufferevent_read(bev, message, sizeof(message));
    printf("服务端返回的数据：%s\n", message);
    printf("请输入需要转换为大写的字符串：\n");
}

// bufferevent 缓冲区的事件回调
void event_cb(struct bufferevent *bev, short events, void *arg){
    if(events & BEV_EVENT_ERROR){
        printf("some other error\n");
        bufferevent_free(bev);
    }
}

void send_cb(evutil_socket_t fd, short what, void *arg){
    char message[message_len];
    struct bufferevent* bev = (struct bufferevent*)arg;
    // 从标准输入读取数据
    int n = read(fd, message, sizeof(message));
    if(n>0){
        bufferevent_write(bev, message, n);
    }
}

int main(){
    // 服务端的ip地址、端口、协议设置
    struct sockaddr_in server_ip_port;
    memset(&server_ip_port, 0, sizeof(server_ip_port));

    server_ip_port.sin_family = AF_INET;
    server_ip_port.sin_port = htons(9311);
    inet_pton(AF_INET, "127.0.0.1", &server_ip_port.sin_addr.s_addr);

    struct event_base* base = event_base_new();
    struct bufferevent* buffer_ev = bufferevent_socket_new(base, -1, BEV_OPT_CLOSE_ON_FREE);

    int re = bufferevent_socket_connect(buffer_ev, (struct sockaddr*)&server_ip_port, sizeof(server_ip_port));
    if(re != 0){
        printf("连接服务端失败：%s\n", strerror(re));
        return -1;
    }

    bufferevent_setcb(buffer_ev, bufferevent_read_data_cb, NULL, event_cb, NULL);

    bufferevent_enable(buffer_ev, EV_READ | EV_PERSIST);
    bufferevent_enable(buffer_ev, EV_WRITE | EV_PERSIST);

    printf("请输入需要转换为大写的字符串：\n");
    // 创建一个事件
    struct event* ev = event_new(base, STDIN_FILENO,
            EV_READ | EV_PERSIST,
            send_cb, buffer_ev);
    event_add(ev, NULL);
    // 开始事件循环监控
    event_base_dispatch(base);

    event_base_free(base);
    return 0;
}
~~~