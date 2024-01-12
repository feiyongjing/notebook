### socker文件进行网络通信
- socker文件进行网络的本质是内核会操作socker文件生成两块缓存区，一个是发送端缓冲区，另一个是接收端缓冲区。然后由一个文件描述符操纵这两个缓冲区。
- 两台主机的分别有2块缓冲区，发送端缓冲区会和远程主机接收端的缓冲区建立连接，这样就形成了两条连接，这样其中任意一台主机都可以通过这两条连接进行发送和接收数据
- 注意网络传输都是使用的大端字节序进行传输，大端字节序是低位的内存地址存放高位的字节，高位的内存地址存放低位的字节

# socker文件操作函数
~~~c
#include <sys/types.h>          
#include <sys/socket.h>

    // 创建 socket 文件描述符
    // domain 参数用于指定套接字的通信协议族，AF_INET 表示IPV4网络协议族，AF_INET6 表示IPV6网络协议族，AF_UNIX 和 AF_LOCAL 表示Unix 域协议族用于本地进程间通信，AF_ISO 表示ISO 协议族
    // type 参数设置协议类型，SOCK_STREAM 表示面向连接的流套接字类型（默认用于TCP）。SOCK_DGRAM 表示无连接的数据报套接字（默认用于UDP）。SOCK_RAW 表示原始套接字，用于直接访问网络层协议，例如 ICMP 和 IGMP
    // protocol 参数设置套接字的协议。通常设置为0，在给定的协议类型中使用默认的协议，当然也可以设置为其他的，例如 IPPROTO_TCP 表示TCP协议、IPPROTO_UDP 表示UDP协议、IPPROTO_ICMP 表示ICMP协议、IPPROTO_IP 表示IP协议
    // 成功返回 socket 文件描述符，即套接字的文件描述符，失败返回-1，并且设置errno
    int socket(int domain, int type, int protocol);

    // 设置 socket 文件描述符，通常用于设置套接字的各种属性，如复用套接字的ip和端口，超时时间、缓冲区大小或特殊的行为选项
    // sockfd 参数是需要设置的套接字文件描述符
    // level 参数设置操作选项的协议，如果是socket协议就是 SOL_SOCKET、TCP设置IPPROTO_TCP，其他级别操作选项，提供控制该选项的适当协议的协议号
    // optname 参数表示要访问的选项名称，例如 SO_REUSEADDR 允许多个套接字绑定到已在使用中的地址（ip地址），SO_REUSEPORT 允许多个套接字绑定到已在使用中的端口（需要 3.9 以上的内核版本）
    // optval 参数是指向包含新选项值的缓冲区的指针，具体的值类型取决于optname 参数，一般结构体或者是整数int，整数int一般是 0 表示启用 optname 参数选项，非 0 表示禁用 optname 参数选项
    // optlen 参数是 optval 参数指向的数据内存大小
    int setsockopt(int sockfd, int level, int optname,
                    const void *optval, socklen_t optlen);


    // socket文件描述符绑定本机IP地址和端口号
    // sockfd 参数是调用 socket 函数后得到的文件描述符，即套接字的文件描述符
    // addr 参数设置本机的 ip 和 port，addr 参数使用的结构体是按照协议族划分的，但是 addr 参数会被强制转换为 sockaddr 的结构体指针，sockaddr 结构体如下
    // addrlen 参数设置 addr 参数占用的内存大小
    // 成功返回零，失败返回-1，并且设置errno
    int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

    // 注意一般都是直接使用 协议族直接对应的结构体，而不是使用 sockaddr 结构体
    // IPv4 协议族对应的结构体是 sockaddr_in，IPv6 协议族对应的结构体是 sockaddr_in6，Unix 域套接字协议族对应的结构体是 sockaddr_un 等
    // sa_family 属性表示的是协议族，有多种协议族不同的宏表示不同的协议族
    // 例如 AF_INET 表示IPV4，AF_INET6 表示IPV6，Unix 域套接字的协议族等
    // sa_data 属性表示的是协议族使用的地址，由于不同的协议族协议不一致但是长度被限制为14个字符之内
    struct sockaddr {
        sa_family_t sa_family;
        char        sa_data[14];
    }

    // 使用 man 7 ip 命令查看 IPv4 协议族对应的结构体 sockaddr_in
    struct sockaddr_in {
        sa_family_t    sin_family; /* IPV4协议族使用 AF_INET */
        in_port_t      sin_port;   /* 网络服务的端口号 */
        struct in_addr sin_addr;   /* ip地址 */
    };

    /* ip地址 */
    struct in_addr {
        /* ip地址，注意这是转换为网络字节序的IP地址，请使用 htonl 函数将long类型本机IP地址转换为网络字节序的IP地址 */ 
        uint32_t       s_addr;     
    };


    // 使用 man 7 unix 命令查看 unix 本地协议族对应的结构体 sockaddr_un
    #define UNIX_PATH_MAX    108
    struct sockaddr_un {
        sa_family_t sun_family;               /* unix 本地协议族使用 AF_UNIX */
        char        sun_path[UNIX_PATH_MAX];  /* 本地文件路径 */
    };


    // 服务端调用 listen 函数将 sucket 套接字由主动态变为被动态，服务端开始监听socket
    // sockfd 参数是调用 socket 函数后得到的文件描述符，即套接字的文件描述符
    // backlog 参数是同时请求的最大连接数
    // 成功返回零，失败返回-1，并且设置errno
    int listen(int sockfd, int backlog);

    // 服务端调用 accept 或 accept4 函数从和客户端已经连接的队列中获得一个连接（内核负责从请求队列中的连接拿到已连接的队列中），并返回一个新的文件描述符，该文件描述符负责与客户端通信
    // accept函数调用若当前没有连接则会阻塞等待，accept4函数调用可以通过flags参数设置是否阻塞
    // sockfd 参数是调用 socket 函数后得到的文件描述符，即套接字的文件描述符
    // addr 参数存储客户端的 ip 和 port
    // addrlen 参数设置 addr 参数占用的内存大小，一般传 NULL
    // flags 参数设置为0 则表示阻塞，SOCK_NONBLOCK表示非阻塞
    // 成功返回一个新的文件描述符，失败返回-1，并且设置errno
    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
    int accept4(int sockfd, struct sockaddr *addr, socklen_t *addrlen, int flags);

    // 客户端调用 connect 函数连接服务端
    // sockfd 参数是调用 socket 函数后得到的文件描述符，即套接字的文件描述符
    // addr 参数设置服务端的 ip 和 port
    // addrlen 参数设置 addr 参数占用的内存大小
    // 成功返回零，失败返回-1，并且设置errno
    int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

    // read和write是文件读写操作函数，参数分别是文件描述符、读写缓冲区、读写字节长度
    // recv和send是socket文件读写操作函数，参数分别是文件描述符、读写缓冲区、读写字节长度、指定发送数据的行为
    // flags参数默认值是0，表示没有特殊行为。MSG_DONTWAIT 表示非阻塞发送，即如果发送缓冲区已满，则立即返回 EAGAIN 错误。MSG_NOSIGNAL 表示在发送数据时，如果对端已经关闭连接，则不会产生 SIGPIPE 信号，而是返回 EPIPE 错误。
    // 成功返回接收到的字节数，返回值为0表示已断开连接，如果发生错误则返回-1。如果发生错误，则设置errno来指示错误。
    // 默认缓冲区如果满了再进行写会阻塞，而缓冲区没有数据是空的再进行读也会阻塞
    #include <unistd.h>
    ssize_t read(int fd, void *buf, size_t count);
    ssize_t write(int fd, const void *buf, size_t count);
    ssize_t recv(int sockfd, void *buf, size_t len, int flags);
    ssize_t send(int sockfd, const void *buf, size_t len, int flags);

    // 关闭 socket 文件描述符
    int close(int fd);

    // recvfrom函数用于UDP协议接收消息
    // 前4个参数和TCP的recv函数一致都是文件描述符、读写缓冲区、读写字节长度、指定发送数据的行为（一般默认是0）
    // src_addr 参数可以获取存储发送方的地址信息
    // addrlen 参数设置 src_addr 参数占用的内存大小
    // 成功返回接收到的字节数，如果发生错误则返回-1。如果发生错误，则设置errno来指示错误。
    ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);

    // sendto函数用于UDP协议发送消息
    // 前4个参数和TCP的send函数一致都是文件描述符、读写缓冲区、读写字节长度、指定发送数据的行为（一般默认是0）
    // dest_addr 参数可以获取存储目的地址信息
    // addrlen 参数设置 dest_addr 参数占用的内存大小
    // 成功返回发送的字节数
    ssize_t sendto(int sockfd, const void *buf, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
~~~
---

# 大小端字节序转换函数
- 如果参数本身就符合转换后的字节序就函数内部是不会进行处理的
~~~c
#include <arpa/inet.h>
    // long类型数据从主机字节序转换网络字节序
    // INADDR_ANY 表示本机任意有效的可用的IP（即存在多网卡多ip的情况任意选择一个ip，并且使用主机字节序表示）
    uint32_t htonl(uint32_t hostlong);

    // short类型数据从主机字节序转换网络字节序
    uint16_t htons(uint16_t hostshort);

    // long类型数据从网络字节序转换主机字节序
    uint32_t ntohl(uint32_t netlong);

    // short类型数据从网络字节序转换主机字节序
    uint16_t ntohs(uint16_t netshort);

    // 将字符串类型的IP地址转化为网络字节序
    // af 参数是协议族类型，IPv4的协议族是 AF_INET
    // src 参数是字符串类型IP地址的内存地址，例如 192.168.0.1 的字符串IP地址
    // dst 参数是网络字节序的内存地址
    // 网络IP地址已成功转换返回1。如果SRC中不包含表示有效网络地址的字符串，则返回0。如果af不包含有效的协议族，则返回-1设置errno
    int inet_pton(int af, const char *src, void *dst);

    // 将网络字节序的IP地址转化为字符串类型的IP地址
    // af 参数是协议族类型，IPv4的协议族是 AF_INET
    // src 参数是网络字节序的IP地址
    // dst 参数是字符串类型IP地址的内存地址，例如 192.168.0.1 的字符串IP地址
    // size 参数是字符串IP地址的长度大小，IPv4是 INET_ADDRSTRLEN ， IPv6 是 INET6_ADDRSTRLEN
    // 成功返回 dst 参数。如果有错误，则返回NULL，并设置errno来指示错误
    const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
~~~
---




