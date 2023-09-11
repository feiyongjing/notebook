## 打开文件
open打开文件返回一个int类型的返回值，返回的是文件描述符，通过文件操作符可以操作文件
文档参考命令：man 2 open
~~~C
用来 打开和创建 一个 文件或设备

#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode)
int creat(const char *pathname, mode_t mode);

open() 通常用于将路径名转换为一个文件描述符 (一个非负的小整数, 在read ,  write 等 I/O操作中将会被使用)，当 open() 调用
       成功, 它会返回一个新的文件描述符 (永远取未用描述符的最小值)，这个调用创建一个新的打开文件, 即分配一个新的独一无二的
       文件描述符,  不会与运行中的任何其他程序共享 (但可以通过 fork(2) 系统调用实现共享)，这个新的文件描述符在其后对
       打开文件操作的函数中使用(参考 fcntl(2))，文件的读写指针被置于文件头
       
参数 flags 是通过 O_RDONLY, O_WRONLY 或 O_RDWR (指明文件是以只读, 只写或读写方式打开的) 与下面的零个或多个可选模式 按位
       -or 操作 得到的:

       O_CREAT
              若文件不存在将创建一个新文件，新文件的属主 (用户ID) 被设置为此程序的有效用户的ID，同样文件所属分组也
              被设置为此程序的有效分组的ID 或者 上层目录的分组ID (这依赖文件系统类型 ,装载选项 和 上层目录 的模式, 参考在
              mount(8) 中描述的 ext2 文件系统的装载选项 bsdgroups 和 sysvgroups )

       O_EXCL 通过 O_CREAT，生成文件，若文件已经存在, 则 open 出错，调用失败，若是存在符号联接 , 将会把它 联接指针 的 指向
              文件忽略 

       O_NOCTTY
              假如 pathname 引用一个 终端设备 — 参考 tty(4) — 即使 进程 没有控制终端 ,这个 终端也不会变成进程的控制终端.

       O_TRUNC
              假如文件已经存在, 且是一个普通文件 ,打开模式又是可写(即文件是用 O_RDWR 或 O_WRONLY 模式打开的) , 就把文件的
              长度设置为零 , 丢弃其中的现有内容，若文件是一个 FIFO 或 终端设备 文件 , O_TRUNC标志被忽略

       O_APPEND
              文件以追加模式打开，在写以前, 文件读写指针被置在文件的末尾 

       O_NONBLOCK 或 O_NDELAY
              打开(open)文件可以以 非阻塞模式打开，此时 文件并没有打开, 也不能使用返回的文件描述符进行后续
              操作, 而是使调用程序等待，此模式是为了 FIFO (命名管道) 的处理, 参考 fifo(4) 这种模式对除了 FIFO 外没有任何
              影响
       
       O_SYNC 打开文件实现I/O 的同步，任何通过文件描述符对文件的 write 都会使调用的进程中断, 直到数据被真正写入硬件中

       O_NOFOLLOW
              假如 pathname 是一个符号联接, 则打开失败，这是 FreeBSD 的 扩充, 从 2.1.126 版本以来被引入到 Linux 中来，从
              glibc2.0.100 库以来, 头文件中包括了这个参数的定义; kernel 2.1.126 以前将忽略它的使用

       O_DIRECTORY
              假如 pathname 不是目录，打开就失败，这个参数是 Linux 特有的, 在 kernel 2.1.126 中 加入 , 为了避免在调用 FIFO 或
              磁带设备时 的 denial-of-service 问题, 但是不应该在执行 opendir 以外使用

       O_LARGEFILE
              在32位系统中支持 大文件系统, 允许打开那些用 31位都不能表示其长度的大文件

       在文件打开后, 这些 flags可选参数 可以通过 fcntl 来改变 

在新文件被创建时, 参数 mode 具体指明了文件使用权限，他通常也会被 umask 修改，所以一般新建文件的权限为 (mode &
       ~umask)，注意模式只被应用于将来对这新文件的使用中; open 调用创建一个新的只读文件, 但仍将返回一个可读写文件
       描述符
       
       mode 只有当在flags中使用 O_CREAT 时才有效, 否则被忽略
       creat 函数相当于 open函数 的参数 flags 等于 O_CREAT|O_WRONLY|O_TRUNC

       后面 是 一些 mode 的 具体 参数:

       S_IRWXU 00700 允许文件的所有者读, 写和执行文件

       S_IRUSR (S_IREAD) 00400 允许文件的所有者读文件

       S_IWUSR (S_IWRITE) 00200 允许文件的所有者写文件

       S_IXUSR (S_IEXEC) 00100 允许文件的所有者执行文件

       S_IRWXG 00070 允许文件所属组读, 写和执行文件

       S_IRGRP 00040 允许文件所属组读文件

       S_IWGRP 00020 允许文件所属组写文件

       S_IXGRP 00010 允许文件所属组执行文件

       S_IRWXO 00007 允许其他用户读, 写和执行文件

       S_IROTH 00004 允许其他用户读文件

       S_IWOTH 00002 允许其他用户写文件

       S_IXOTH 00001 允许其他用户执行文件

RETURN VALUE 返回值
       open  和  creat  都返回一个新的文件描述符 (若是有错误发生返回 -1，并在 errno 设置错误信息) 注意 open 可以打开设备专用
       文件, 但是 creat 不能创建,需要用 mknod(2)来代替
       若文件是新建立的, 他的 atime(上次访问时间), ctime(创建时间), mtime(修改时间) 都被修改为当前时间, 上层目录的atime,
       ctime 也被同样修改

ERRORS 错误信息
       EEXIST 参数 O_CREAT and O_EXCL 被使用，但是文件( pathname )已经存在

       EISDIR 文件名( pathname ) 是 一个 目录 , 而又涉及到写操作

       EACCES 访问请求不允许(权限不够)，在文件名( pathname )中有一目录不允许搜索(没有执行权限)，或者文件还不存在且对
              上层目录的写操作又不允许

       ENAMETOOLONG 文件名( pathname )太长了

       ENOENT 目录( pathname )不存在或者是一个悬空的符号联接

       ENOTDIR pathname不是一个子目录

       ENXIO  使用 O_NONBLOCK | O_WRONLY, 命名的文件是 FIFO , 所读文件还没有打开的文件, 或者打开一个设备专用文件而相应
              的设备不存在

       ENODEV 文件( pathname )引用了一个设备专用文件, 而相应的设备又不存在，(这是linux kernel 的 一个bug - ENXIO 一定会被返回)

       EROFS  文件( pathname )是一个只读文件, 写操作被拒绝

       ETXTBSY 文件( pathname )是一个正在被执行的可执行文件，又有写操作被请求

       EFAULT pathname 在一个你不能访问的地址空间

       ELOOP  在分解 pathname 时，遇到太多符号联接 或者 指明 O_NOFOLLOW 表示 pathname 是一个符号联接

       ENOSPC pathname 将要被创建，但是设备又没有空间储存 pathname 文件了

       ENOMEM 可获得的核心内存(kernel memory) 不够

       EMFILE 程序打开的文件数已经达到最大值了

       ENFILE 系统打开的总文件数已经达到了极限
~~~
---
 

## 关闭文件
当一个进程终止时，该进程打开的文件如果没有关闭那么内核会自动关闭打开的文件。但是通常情况下程序运行没有意外情况下是不会终止，这时需要手动关闭不再使用的文件以免造成系统资源的浪费
文档参考命令：man 2 close
~~~C
close - 关闭一个文件描述符

SYNOPSIS 总览
       #include <unistd.h>

       int close(int fd);

DESCRIPTION 描述
       close 关闭一个文件描述符，使它不指向任何文件和可以在新的文件操作中被再次使用，任何与此文件相关联的以及
       程序所拥有的锁，都会被删除(忽略那些持有锁的文件描述符)

       假如 fd 是 最后一个文件描述符与此资源相关联，则这个资源将被释放. 若此描述符是最后一个引用到此文件上的，则
       文件将使用 unlink(2) 删除.

RETURN VALUE 返回值
       close 返回 0 表示成功，或者 -1 表示有错误发生

ERRORS 错误信息
       EBADF  fd 不是 一个有效 的已被打开的文件的描述符

       EINTR  The close() 调用被一信号中断

       EIO    I/O 有错误发生

NOTES 注意
       通常不检测返回值，除了发生严重的程序错误，文件系统使用了 "write-behind" 的技术提高了执行 write(2) 时的性能，即使
       还没有被写，写操作也会成功，错误信息在写操作以后报告，但是这保证在关闭文件时报告，在关闭文件时不检测返回值
       可能会导致数据的丢失，这一点在NFS 和 磁盘配额上比较明显.

       由于内核会延迟写，所以就算成功关闭一个文件不能保证数据被成功的写到磁盘上，当文件流关闭时，对文件系统来说
       一般不去刷新缓冲区，如果你要保证数据写入磁盘等物理存贮器中就使用 fsync(2) 或 sync(2)，他们会做到你想做的(对于
       这一点要依赖磁盘设备).

~~~
---
 

## 从文件描述符中读取数据
文档参考命令：man 2 read
~~~C
read - 在文件描述符上执行读操作

概述
       #include <unistd.h>

       ssize_t read(int fd, void *buf, size_t count);

描述
       read() 从文件描述符 fd 中读取 count 字节的数据并放入从 buf 开始的缓冲区中.

       如果 count 为零,read()返回0,不执行其他任何操作，如果 count 大于SSIZE_MAX,那么结果将不可预料.

返回值
       成功时返回读取到的字节数(为零表示读到文件描述符)，此返回值受文件剩余字节数限制.当返回值小于指定的字节数时
       并不意味着错误;这可能是因为当前可读取的字节数小于指定的字节数(比如已经接近文件结尾,或者正在从管道或者终端读取数据,或者
       read()被信号中断)，发生错误时返回-1，并置 errno 为相应值.在这种情况下无法得知文件偏移位置是否有变化.

错误代码
       EINTR  在读取到数据以前调用被信号所中断

       EAGAIN 使用 O_NONBLOCK 标志指定了非阻塞式输入输出,但当前没有数据可读

       EIO    输入输出错误.可能是正处于后台进程组进程试图读取其控制终端,但读操作无效,或者被信号SIGTTIN所阻塞,
              或者其进程组是孤儿进程组，也可能执行的是读磁盘或者磁带机这样的底层输入输出错误

       EISDIR fd 指向一个目录

       EBADF  fd 不是一个合法的文件描述符，或者不是为读操作而打开

       EINVAL fd 所连接的对象不可读

       EFAULT buf 超出用户可访问的地址空间

       也可能发生其他错误,具体情况和 fd 所连接的对象有关，POSIX 允许 read 在读取了一定量的数据后被信号所中断,并返回 -1(且 errno
       被设置为EINTR)，或者返回已读取的数据量
~~~
---
 

## 向文件描述符写入数据
文档参考命令：man 2 write
~~~C
write -在一个文件描述符上执行写操作

概述
       #include <unistd.h>

       ssize_t write(int fd, const void *buf, size_t count);

描述
       write 向文件描述符 fd 所引用的文件中写入从 buf 开始的缓冲区中 count 字节的数据，POSIX规定,当使用了write()之后再使用
       read(),那么读取到的应该是更新后的数据，但请注意并不是所有的文件系统都是 POSIX兼容的.

返回值
       成功时返回所写入的字节数(若为零则表示没有写入数据)，错误时返回-1,并置errno为相应值
       若count为零,对于普通文件无任何影响,但对特殊文件 将产生不可预料的后果

错误代码
       EBADF  fd 不是一个合法的文件描述符或者没有以写方式打开

       EINVAL fd 所指向的对象不可写

       EFAULT buf 不在用户可访问地址空间内

       EPIPE  fd 连接到一个管道,或者套接字的读方向一端已关闭.此时写进程 将接收到 SIGPIPE 信号;如果此信号被捕获,阻塞或忽略,那么将返回错误
              EPIPE

       EAGAIN 读操作阻塞,但使用 O_NONBLOCK 指定了非阻塞式输入输出

       EINTR  在写数据以前调用被信号中断

       ENOSPC fd 指向的文件所在的设备无可用空间

       EIO    当编辑一个节点时发生了底层输入输出错误

       可能发生了其他错误,取决于 fd 所连接的对象
~~~
---
 

## 移动文件内容指针
文档参考命令：man 2 lseek
~~~C
#include <sys/types.h>
#include <unistd.h>
off_t lseek(int filedes, off_t offset, int whence) ;

打开的每个文件都有一个与其相关联的“当前文件位移量”。它是一个非负的整数，用以度量从文件开始处计算的字节数。
通常，读、写操作都从当前文件位移量处开始，并使位移量增加所读或写的字节数。按系统默认，当打开一个文件时，
除非指定O_A P P E N D选择项，否则该位移量被设置为0。

第一个参数是文件描述符；
第二个参数是偏移量，int型的数，正数是向后偏移，负数向前偏移；
第三个参数指从哪里开始偏移是有三个选项：
1.SEEK_SET:文件内容的开始指针
2.SEEK_CUR:当前的偏移量位置，其实就是上次读文件或者是写文件后指针的移动位置
3.SEEK_END:文件内容的结束指针  

如果文件内容指针移动到了文件内容结束指针之后，不进行写操作是不会扩展文件空间的，但是一旦有写操作就会立刻在文件中填充一段数据表示空间占用，并在文件内容指针之后写入数据，这样做的原因是提前判断磁盘空间是否足够填入数据，比如下载数据时检查磁盘是否有空间存放文件就是这么做的

调用成功完成后，lseek()返回结果偏移量位置，从文件开始以字节计，可以用来获取文件大小。发生错误，返回值是 -1, errno设置为错误信息

~~~
---
 

## 获取文件状态信息
文档参考命令：man 2 stat
~~~C
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

   int stat(const char *pathname, struct stat *buf);
   int fstat(int fd, struct stat *buf);
   int lstat(const char *pathname, struct stat *buf);
   
   # 获取文件的属性放入stat结构体中，参数pathname是文件路径指针，fd是文件描述符、buf是存放文件属性的结构体指针
   # lstat()与stat()的不同之处在于，如果pathname是一个符号链接，那么lstat()返回关于链接本身的信息，而不是
它所引用的文件
   # stat()与fstat()只是参数有所不同，功能是一样的
   # 返回0表示成功，-1表示失败

#include <fcntl.h>           /* Definition of AT_* constants */
#include <sys/stat.h>

   int fstatat(int dirfd, const char *pathname, struct stat *buf, int flags);
   
   
   
# stat结构体信息
struct stat {
   dev_t     st_dev;         /* 文件的设备ID，如果不是设备文件是没有这个ID的*/
   ino_t     st_ino;         /* 文件的inode */
   mode_t    st_mode;        /* 文件类型和权限 */
   nlink_t   st_nlink;       /* 硬链接数 */
   uid_t     st_uid;         /* 所有者的用户ID */
   gid_t     st_gid;         /* 用户组ID */
   dev_t     st_rdev;        /* 设备ID(如果特殊文件)，如果不是设备文件是没有这个ID的 */
   off_t     st_size;        /* 总大小，以字节为单位 */
   blksize_t st_blksize;     /* 文件系统I/O块大小，每个块是512字节 */
   blkcnt_t  st_blocks;      /* 文件占用的块数量 */

   /* 从Linux 2.6开始，内核支持纳秒，以下时间戳字段的精度。*/
   struct timespec st_atim;  /* 最后一次访问时间 */
   struct timespec st_mtim;  /* 上次修改时间 */
   struct timespec st_ctim;  /* 最后一次状态更改时间 */

   #define st_atime st_atim.tv_sec      /* 向后兼容性 */
   #define st_mtime st_mtim.tv_sec
   #define st_ctime st_ctim.tv_sec
};
   
#  st_mode字段的文件模式组件定义的

           S_ISUID     04000   只针对文件，无法用于目录，指定的用户拥有和文件所有者一样的执行权限
           S_ISGID     02000   只针对文件，无法用于目录，指定的用户组拥有和文件所有组一样的执行权限
           S_ISVTX     01000   只针对目录，无法用于文件，普通用户对目录有wx权限但是有粘着位的情况下，只能删除目录下文件的所属组是自己的文件或目录

           S_IRWXU     00700   所有者具有读、写和执行权限
           S_IRUSR     00400   所有者具有读权限
           S_IWUSR     00200   所有者具有写权限
           S_IXUSR     00100   所有者具有执行权限

           S_IRWXG     00070   组有读、写和执行权限
           S_IRGRP     00040   组有读权限
           S_IWGRP     00020   组有写权限
           S_IXGRP     00010   组具有执行权限

           S_IRWXO     00007   其他有读、写和执行权限
           S_IROTH     00004   其他用户有读权限
           S_IWOTH     00002   其他用户有写权限
           S_IXOTH     00001   其他具有执行权限

# st_mode字段进行文件权限判断
if (sb.st_mode & S_IRUSR) {
    /* 所有者具有读权限 */
}
#注意读写执行的7权限判断
if （(sb.st_mode & S_IRUSR) && (sb.st_mode & S_IWUSR) && (sb.st_mode & S_IXUSR)） {
    /* 所有者具有读、写和执行权限 */
}          

# st_mode字段进行文件类型判断
# 不同的文件类型结果是不同的宏
# S_ISSOCK 0140000  套接字(POSIX.1-1996中没有)
# s_islink 0120000  符号链接(POSIX.1-1996中没有)
# S_ISREG  0100000  普通文件
# S_ISBLK  0060000  块设备
# S_ISDIR  0040000  目录
# S_ISCHR  0020000  字符设备
# S_ISFIFO 0010000  FIFO(命名管道，管道文件)
# 方式一：直接使用内置的宏函数判断
stat(pathname, &sb);
if (S_ISREG(sb.st_mode)) {
   /* 是一个普通文件 */
}
# 方式二：和宏S_IFMT（0170000 文件类型位字段的掩码）相与得出结果
stat(pathname, &sb);
if ((sb.st_mode & S_IFMT) == S_IFREG) {
    /* 是一个普通文件 */
}

~~~
---
 
## 错误信息查看
错误信息文档参考命令：man errno  可以看到错误信息对应的宏
~~~C
#include <errno.h>

perror("错误的信息是")

调用perror系统函数会在控制台打印 错误的信息是：具体的错误信息

~~~
---
