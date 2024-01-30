

# select 多路IO复用，man select
~~~c
#include <sys/select.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>


// select 函数设置内核监听文件描述符的读、写、异常事件，注意最多只能监听1024个文件描述符，如果需要更多需要修改内核源码的宏 FD_SETSIZE=1024 为更大然后编译内核源码
// nfds 参数设置监听文件描述符的范围，一般是最大的文件描述符+1，表示监听当前进程的全部文件描述符范围
// readfds 参数设置监听那些文件描述符集合的读事件，并且后续可以通过判断文件描述符集合中是否存在指定文件描述符，知道指定文件描述符是否有读事件，后续通过读取指定文件描述符内容后，文件描述符集合中就不存在指定文件描述符了
// writefds 参数设置监听那些文件描述符集合的写事件，并且后续可以通过判断文件描述符集合中是否存在指定文件描述符，知道指定文件描述符是否有写事件
// exceptfds 参数设置监听那些文件描述符集合的异常事件，并且后续可以通过判断文件描述符集合中是否存在指定文件描述符，知道指定文件描述符是否有异常事件
// timeout 参数设置监听超时时长。设置为NULL表示调用函数一直阻塞直到有监听到文件描述符的读、写、异常事件会立刻返回。设置为0表示不阻塞，不管是否有监听到文件描述符的读、写、异常事件都会立刻返回。设置大于0表示在该时长内阻塞，直到有监听到文件描述符的读、写、异常事件立刻返回 或者是超过该时长也会立刻返回
// 返回值是产生事件的文件描述符数量
int select(int nfds, fd_set *readfds, fd_set *writefds,
            fd_set *exceptfds, struct timeval *timeout);

struct timeval {
    time_t      tv_sec;         /* 秒 */
    suseconds_t tv_usec;        /* 微秒 */
};

// 从文件描述符集合中移除指定的文件描述符
void FD_CLR(int fd, fd_set *set);

// 判断文件描述符集合中是否存在指定文件描述符
int  FD_ISSET(int fd, fd_set *set);

// 向文件描述符集合添加指定的文件描述符
void FD_SET(int fd, fd_set *set);

// 清空文件描述符集合
void FD_ZERO(fd_set *set);
~~~
---


# poll 多路IO复用，man poll
### 优点是比 select 监听的文件多，缺点是对于一些操作系统不支持
~~~c
#include <poll.h>


// poll 函数设置内核监听文件描述符的读、写、异常事件，注意监听监听的文件描述符数量是 /proc/sys/fs/file-max 文件记录的数值，如果需要修改请编辑 vim /etc/security/limits.conf 文件的如下配置，soft 和 hard 分别是 ulimit 命令可以修改的最小和最大限制
// * soft nofile 1024
// * hard nofile 655350
// fds 参数设置监听那些文件描述符集合（注意fds 参数结构体数组的首地址）的读、写、异常事件，并且后续可以通过判断文件描述符集合中是否存在指定文件描述符，知道指定文件描述符是否有读、写、异常事件，后续通过读取指定文件描述符内容后，文件描述符集合中就不存在指定文件描述符了
// nfds 参数设置监听文件描述符的范围，一般是最大的文件描述符+1，表示监听当前进程的全部文件描述符范围
// timeout 参数设置监听超时时长。设置为 -1 表示调用函数一直阻塞直到有监听到文件描述符的读、写、异常事件会立刻返回。设置为0表示不阻塞，不管是否有监听到文件描述符的读、写、异常事件都会立刻返回。设置大于0表示在该时长内阻塞，直到有监听到文件描述符的读、写、异常事件立刻返回 或者是超过该时长也会立刻返回
// 返回值是产生事件的文件描述符数量，如果返回 -1 表示异常错误
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

// 文件描述符监控
struct pollfd {
    int   fd;         /* 监控的文件描述符，如果是 -1 表示不监控 */
    short events;     /* 需要监控的事件，读、写、异常事件，POLLIN 监听读事件、POLLOUT 监听写事件、POLLERR 监听异常事件 */
    short revents;    /* 内核在监控结束后告诉程序文件描述符进行过那些事件 */
};

~~~
---


# epoll 多路IO复用，man epoll_create
### 优点是比 select 监听的文件多，缺点是对于一些操作系统不支持
~~~c
#include <sys/epoll.h>

// epoll_create 函数创建一棵epoll树（红黑树，即平衡二叉树）
// size 参数是树的大小，但是其实现在的系统内核版本使用一般传递大于0 的数即可
// 返回 epoll 树的根节点
int epoll_create(int size);

// epoll_ctl 函数将文件描述符当成节点放入 epoll 树监控、修改 epoll 树上的节点、删除 epoll 树上的节点
// epfd 参数是 epoll 树的根节点
// op 参数设置 epoll 树的操作，EPOLL_CTL_ADD 是添加文件描述符到epoll树，EPOLL_CTL_MOD 是修改 epoll 树上的文件描述符，EPOLL_CTL_DEL 是删除 epoll 树上的节点
// fd 参数是需要操作的文件描述符节点
// event 参数是设置监控文件描述符的事件
// 成功返回零，发生错返回-1，并设置errno
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

typedef union epoll_data {
    void        *ptr;         /* 一个自定义的结构体数据，通常该结构体中存在监控的文件描述符、监控事件、以及事件发生时的回调函数（该回调函数在 epoll_wait 监听事件获取到后，进行手动调用）。如下有例子 xx_events 自定义结构体 */
    int          fd;          /* 内核监控到发生事件的文件描述符 */
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* epoll 监控事件，EPOLLIN 是监控读事件、EPOLLOUT 是监控写事件、EPOLLERR 是监控异常错误事件、 EPOLLET 是epoll 的ET模式*/
    epoll_data_t data;        /* 委托内核监控的数据 */
};

// 自定义的结构体数据，设置事件回调以便在后续手动调用回调函数
struct xx_events{
    int          fd;          /* 产生事件文件描述符 */
    int          events;      /* 监听到的事件 */
    void (*call_back)(int fd, int events, void* args);  /* 事件产生回调函数，一般都是将产生事件文件描述符、监听到的事件当成参数，并且设置void* args 参数传递一些事件回调所需要的参数 */
}


// epoll_wait 函数调用会开始监控 epoll 树上的节点事件
// epfd 参数是 epoll 树的根节点
// events 参数是 一个结构体数组的首地址指针，该结构体会在函数返回时存放监听到的文件描述符事件信息
// maxevents 参数是 events 参数的数组长度
// timeout 参数设置监听超时时长。设置为 -1 表示调用函数一直阻塞直到有监听到文件描述符的读、写、异常事件会立刻返回。设置为0表示不阻塞，不管是否有监听到文件描述符的读、写、异常事件都会立刻返回。设置大于0表示在该时长内阻塞，直到有监听到文件描述符的读、写、异常事件立刻返回 或者是超过该时长也会立刻返回
// 返回值是产生事件的文件描述符数量，如果返回 -1 表示异常错误
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);


/*  epoll 的 LT模式 和 ET模式 （默认是LT模式）
    LT模式：如果在 epoll_wait 通知有读事件去读取对应文件描述符在内核缓冲区中数据没有一次性的读取完，在下一次 epoll_wait 调用时还会通知该文件描述符有读事件
    ET模式：如果在 epoll_wait 通知有读事件去读取对应文件描述符在内核缓冲区中数据没有一次性的读取完，在下一次 epoll_wait 调用时不会会通知该文件描述符有读事件，除非该文件描述符对应的内核缓冲区数据发送变化（例如继续写入数据）后，下一次 epoll_wait 调用时才会通知该文件描述符有读事件
 */
~~~
---