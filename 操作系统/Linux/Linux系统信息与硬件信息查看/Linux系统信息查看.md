~~~shell
# 查看系统开机时内核检测信息
dmesg
# 显示的信息太多了，一般使用dmesg | grep CPU查看cpu相关信息，使用dmesg | grep eth0查看网卡信息

# 查看cpu的详细信息，注意/proc/目录是内存的挂载点，也就是说cup的详细信息在内存中，原因是cpu可能会在断电后更换
cat /proc/cpuinfo

# 查看当前操作系统的发行版本，有些命令不行
lsb-release -a
cat /etc/centos-release

# 查看系统和内核相关信息
uname 
# -a参数查看系统相关所有信息：当前系统的内核名称、主机名、内核发行版本、节点名、压制时间、硬件名称、硬件平台、处理器类型以及操作系统名称等信息
# -r参数只查看内核版本
# -s参数查看内核名称

# 查看系统是32位还是64位
file /bin/ls
# 只要参数是一个非系统内置命令就可以查看到

# 显示主机信息
hostnamectl

# 设置主机名称，需要root权限
hostnamectl set-hostname [新的主机名称]

# 查看当前时间
date

# 查看当前时区设置
timedatectl

# 显示所有可用的时区
timedatectl list-timezones                                                                                   

# 设置当前时区，需要root权限
timedatectl set-timezone Asia/Shanghai
timedatectl set-time YYYY-MM-DD
timedatectl set-time HH:MM:SS

# 查看当前年份当前月份日历
cal
# -y参数显示当前年份全部月份日历

# 显示当前用户的资源限制
ulimit -a
# 资源单位后面是改资源大小的修改参数，unlimited表示无限大
# core file size（核心文件大小）：当进程崩溃时，系统可以生成一个核心文件，这个限制决定了核心文件的最大大小。
# data seg size（数据段大小）：进程的最大数据段大小。
# scheduling priority（调度优先级）：进程的调度优先级。
# file size（文件大小）：进程能创建的最大文件大小。
# pending signals（待处理信号）：进程可排队的最大信号数量。
# max locked memory（最大锁定内存）：进程可锁定在内存中的最大字节数。
# max memory size（最大内存大小）：进程的最大虚拟内存大小。
# open files（打开文件）：进程可打开的最大文件数量。
# pipe size（管道大小）：管道缓冲区的大小。
# stack size（栈大小）：进程堆栈的最大大小。
# cpu time（CPU时间）：进程可使用的最大CPU时间。
# max user processes（最大用户进程数）：用户可运行的最大进程数。
# virtual memory（虚拟内存）：进程的最大虚拟内存大小。

~~~