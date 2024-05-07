### 主目录：/root、/home/username

/usr目录是指 unix system resource 即unix系统资源目录

用户可执行文件：/bin、/usr/bin、/usr/local/bin   
一些用户的常用命令存放目录: 其中/bin是存放重要的可执行命令、/usr/bin存放不太重要如图形环境或Office工具、/usr/local/bin是存放源码包安装的软件

系统可执行文件：/sbin、/usr/sbin、/usr/local/sbin   
root用户的常用命令存放目录：其中/sbin是存放重要的可执行命令、/usr/sbin存放不太重要如图形环境或Office工具、/usr/local/sbin是存放源码包安装的软件

共享库：/lib、/usr/lib、/usr/local/lib
/lib是存放重要的库是系统需要使用的库、/usr/lib存放不太重要的库如图形环境或Office工具使用的库、/usr/local/lib是存放源码包安装的软件需要的库

其他挂载点：/media、/mnt
/media是默认提供的设备挂载点，比如光驱
/mnt是手动挂载点

配置文件目录：/etc
一些软件的安装位置（比如docker）：/opt
系统的头文件位置：/usr/include

设备文件：/dev
/dev/input目录下是存放各种设备的输入

临时文件：/tmp
一些程序将一些不重要的临时文件存放在tmp中

内核和Bootloader：/boot

服务器数据：/var、/srv
/var/run存放程序的pid，/var/log存放日志信息，/var/lib存放应用程序的持久数据，例如数据库文件、Web 服务器的文件等

### 系统信息：/proc、/sys
- /proc中的文件目录是存在于内存中的，里面存放了系统核心、外部设备、网络状态、进程相关的信息
- ps、top、free、lscpu等命令都是从这个目录下获取的信息
- 