# centOS6系统日志 
~~~shell
# 系统的日志服务是rsyslogd 查看系统日志服务是否开启
ps aux | grep rsyslogd
systemctl status rsyslog

# /var/log/目录保存系统的默认日志
# /var/log/cron 记录系统定时任务相关的日志
# /var/log/cups 记录打印信息的日志
# /var/log/dmesg 记录系统开机内核自检的日志，也可以使用dmesg命令查看这一部分日志
# /var/log/btmp  记录错误登录系统的日志，这个日志是二进制文件不能直接使用vim查看需要使用lastb查看
# /var/log/lastlog  记录系统中所有用户最后一次登录时间的日志，这个日志是二进制文件不能直接使用vim查看需要使用lastlog查看
# /var/log/mailog  记录系统中邮件信息
# /var/log/messages  记录系统中重要信息，这个日志文件记录了系统在绝大多数重要信息，系统出现问题时首先就要检查它
# /var/log/secure  记录验证和授权的信息，只要涉及账号和密码的程序都会记录。比如系统登录，ssh远程登录，su切换用户，sudo授权，用户的添加和修改密码等
# /var/log/wtmp   永久记录所有用户的登录、注销信息。同时记录系统的启动、重启、关机事件这个日志是二进制文件不能直接使用vim查看需要使用last查看
# /var/log/utmp   记录当前已经登录的用户信息。这个日志是二进制文件不能直接使用vim查看需要使用w、who、users命令来查看
# rpm包安装的服务日志也有一些在/var/log/目录下

# 一般的系统日志文件基本的日志格式如下
# 事件产生的时间
# 发生事件的服务器主机名
# 产生事件的服务名称或程序的名称
# 事件的具体信息

# 日志配置文件/etc/rsyslog.conf
# 例如authpriv.*    /var/log/secure
# authpriv是服务的名称  .代表连接符  *通配符表示日志的等级   /var/log/secure路径表示服务的日志记录在该路径
# 服务的名称有很多例如 authpriv表示认证相关服务、mail表示邮箱服务、cron表示系统定时任务服务、kern表示内核服务
# 连接符有很多例如 .代表记录后面所表示的日志等级以及高于该日志等级的日志， .= 表示只记录后面所表示的日志等级而其他的日志等级日志不记录 .!表示不记录后面所表示的日志等级而其他的日志等级日志都记录
# 日志的等级从低到高：debug调试等级、info基本信息通知、notice普通信息（具有一定的重要性）、warning警告信息（但是还不到影响服务和系统运行的地步）、err错误信息（一般到err就会影响服务和系统的运行了）、crit临界状态（比err等级严重）、alert警告状态信息（比crit等级严重必须立即采取行动）、emerg疼痛等级信息(系统以及无法使用了)
# 日志的记录位置设置有几种例如直接使用本机的绝对路径、记录到设备文件中如/dev/lp0打印、转发给远程的主机如@192.168.0.23:3000、发送到用户的邮箱root、忽略丢弃日志如~等

# 日志的轮替配置/etc/logrotate.conf，配置参数如下
# daily   表示日志每天都会轮替，即创建新的日志文件记录日志
# weeky   表示日志每周都会轮替，即创建新的日志文件记录日志
# monthly   表示日志每月都会轮替，即创建新的日志文件记录日志
# rotate 数字  表示轮替时会保留的日志文件个数。0代表没有备份日志
# compress     表示日志轮替时，旧的日志需要进行压缩
# create mode owner group  表示建立新日志时指定新的日志的权限与所有者和所属组。如 create 0600 root utmp
# mail address  当日志轮替时，输出内容通过邮件发送到指定的邮件地址，如 mail feiyongjing@cloudgrow.cn
# missingok     如果日志不存在，则忽略该日志的警告信息
# notifempty    如果日志为空文件，则不进行日志轮替
# minsize 大小  日志轮替的最小值。就是表示日志文件的大小一定到达这个最小才会轮替，否则就算时间达到也不轮替
# size 大小     日志只有大于指定大小才会进行日志轮替，而不是按照时间轮替。例如size 100k
# dataext       使用日期作为日志轮替文件的后缀。如messages-20221211
# 对于一些特殊的日志文件使用 {} 包裹特殊轮替配置 {}中的配置如果有与{}之外的配置重叠会覆盖{}之外的配置
# 注意日志的轮替不是到达时间或者是到达日志轮替的大小后就立即轮替而是挑选系统的空闲时间进行日志轮替
# 一般来说rpm包安装的日志是不需要进行设置日志的轮替的，使用默认的轮替就足够了，但是源码包安装的服务就需要手动配置日志的轮替

# 查看日志轮替情况和强制日志轮替
logrotate [参数] /etc/logrotate.conf
# -v参数查看日志轮替情况和过程
# -f参数强制进行日志轮替，不管是否到达轮替的时间和日志轮替的大小

~~~
---

# centOS7查看所有服务的日志
~~~shell
# centOS7的Systemd 统一管理所有 Unit 的启动日志。带来的好处就是，可以只用journalctl一个命令，查看所有日志（内核日志和应用日志）。日志的配置文件是/etc/systemd/journald.conf。journalctl功能强大，用法非常多。基本所有的journalctl命令都需要root权限执行查看日志

# 查看所有日志（默认情况下 ，只保存本次启动的日志）
journalctl

# 查看内核日志（不显示应用日志）
journalctl -k

# 查看系统本次启动的日志
journalctl -b
journalctl -b -0

# 查看上一次启动的日志（需更改设置）
journalctl -b -1

# 查看指定时间的日志
journalctl --since="2012-10-30 18:17:16"
journalctl --since "20 min ago"
journalctl --since yesterday
journalctl --since "2015-01-10" --until "2015-01-11 03:00"
journalctl --since 09:00 --until "1 hour ago"

# 显示尾部的最新10行日志
journalctl -n

# 显示尾部指定行数的日志
journalctl -n 20

# 实时滚动显示最新日志
journalctl -f

# 查看指定服务的日志
journalctl /usr/lib/systemd/systemd

# 查看指定进程的日志
journalctl _PID=1

# 查看某个路径的脚本的日志
journalctl /usr/bin/bash

# 查看指定用户的日志
journalctl _UID=33 --since today

# 查看某个 Unit 的日志
journalctl -u nginx.service
journalctl -u nginx.service --since today

# 实时滚动显示某个 Unit 的最新日志
journalctl -u nginx.service -f

# 合并显示多个 Unit 的日志
journalctl -u nginx.service -u php-fpm.service --since today

# 查看指定优先级（及其以上级别）的日志，共有8级
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug
journalctl -p err -b

# 日志默认分页输出，--no-pager 改为正常的标准输出
journalctl --no-pager

# 以 JSON 格式（单行）输出
journalctl -b -u nginx.service -o json

# 以 JSON 格式（多行）输出，可读性更好
journalctl -b -u nginx.serviceqq
 -o json-pretty

# 显示日志占据的硬盘空间
journalctl --disk-usage

# 指定日志文件占据的最大空间
journalctl --vacuum-size=1G

# 指定日志文件保存多久
journalctl --vacuum-time=1years 

~~~
---
 