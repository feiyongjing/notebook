
~~~shell
# 查看系统中所有的用户登录后的默认shell，统计每种默认shell的用户数量
awk -F: '{print $7}' /etc/passwd | sort | uniq -c

# 查看用户登录后的默认shell是bash的用户，检查是否有不在预期情况的非法用户，即被别人攻击留下了后门
cat /etc/passwd | grep bash

# 查看用户id和用户组id，分析是否有非法权限，主要是其他用户是否用户id和组id是root用户和组表示的0
sudo id [username]

# 列出所有用户最后一次的登录时间，分析是否有非法用户、非法时间、非法ip地址远程登录
sudo lastlog [-u username]
# -u参数指定用户的名称，表示这查看这个用户的信息

# 列出所有用户的登录历史，分析是否有非法用户、非法时间、非法ip地址远程登录
sudo last [username]
# username指定用户的名称，表示这查看这个用户的登录历史

# 列出所有用户登录失败历史，分析是否有非法用户、非法时间、非法ip地址远程登录
sudo lastb [username]
# username指定用户的名称，表示这查看这个用户的登录失败历史

# 检查/etc/sudoers 文件和 /etc/sudoers.d/ 目录下的文件内容，分析是否有用户或者用户组非法获得sudo权限
cat /etc/sudoers

# 检查系统中是否有可疑服务设置为开机启动
systemctl list-unit-files | grep enabled

# 系统的运行级别是3，就检查/etc/rc3.d/ 目录下的软连接文件（指向真实的程序），其中软链接文件开头是S的文件是开机启动的服务程序，开头是K的文件是关机时执行的程序
sudo ls /etc/rc3.d/ -l 
# 查看系统的运行级别 runlevel 如果是其他的系统运行级别就检查其他的 /etc/rc[n].d/ 目录下的软连接文件（指向真实的程序）
# 检查有没有可疑路径的程序，或者是程序的大小不太对劲，即被黑客替换了程序并且伪装成正常的程序

# 检查arp缓存，是否有ip地址与mac地址不匹配，防止中间人攻击
arp

# 检查是否有非法的定时任务，检查 /etc/crontab 和 /etc/cron.daily/目录、/etc/cron.hourly/目录、/etc/cron.monthly/目录、/etc/cron.weekly/ 目录下文件中是否有定时任务
cat /etc/crontab
crontab -l # 检查现在的定时任务中是否有非法的定时任务

# 不同的linux发行版使用的默认shell是不同的，不同的shell本质是用户登录时shell的初始化启动默认执行的配置文件（环境变量配置文件）不同
# 检查以下文件是否有恶意代码或者命令，防止用户登录时shell的初始化启动配置文件执行其中的恶意代码或者命令
# /etc/profile  
# /etc/profile.d/*.sh
# ~/.bash_profile
# ~/.profile
# ~/.bashrc
# /etc/bashrc

# 检查历史执行的命令确保没有非法命令执行
# 临时执行 export HISTTIMEFORMAT=%F %T 或者将命令写入环境变量配置文件中应用后，history 可以查看命令和命令执行的具体时间
history

# 查看当前登录的用户详细信息，检查是否有非法用户已经登录
w
# 显示的第一行（可以使用uptime命令只查看第一行），例如 12:05:13 up 18 days,  1:28,  2 users,  load average: 1.27, 1.35, 1.42
# 12:05:13 up 18 days,  1:28 表示当前系统的使用以及该系统正常运行了18天1小时28分钟
# 2 users 表示当前有两个用户登录
# load average: 1.27, 1.35, 1.42 分别表示过去1分钟、5分钟、15分钟系统cpu和内存结合的负载情况，数字越大则负载越大
# 第一行之外的内容有8列，前四列的信息与who命令看到的一致
# IDLE显示的shell连接后的闲置时间，即连接后没有操作的时间
# JCPU显示累计占用cpu的时间
# PCPU显示最后一次执行的操作占用cpu的时间
# WHAT显示最后一次执行的操作

# 检查/var/log目录下的日志，是否有异常日志

# 通过top命令查看是否有可疑进程，通过以下命令进一步检查进程是否非法
cat /proc/<PID>/cmdline                                         # 查看指定pid进程的启动命令
readlink /proc/[pid]/cwd                                        # 查看指定pid进程启动的目录
cat /proc/[pid]/environ | tr '\0' '\n' | sed 's/^[^=]*=//'      # 查看指定pid进程启动时的完整环境变量
lsof -p [pid]                                                   # 查看指定pid进程打开的文件
~~~