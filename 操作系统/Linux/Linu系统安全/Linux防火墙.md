# Linux防火墙
Linux 系统下的防火墙主要分为两层：第一层是对 IP 进行过滤的 iptables，第二层就是 tcp_wrappers

# iptables防火墙
iptables 是集成在 Linux 内核中的包过滤防火墙系统。
使用 iptables 可以添加、删除具体的过滤规则，iptables 默认维护着 4 个表和 5 个链，所有的防火墙策略规则都被分别写入这些表与链中。


## “五链”是指内核中控制网络的 NetFilter 定义的 5 个规则链。
- 5 个规则链：INPUT（入站数据过滤）、OUTPUT（出站数据过滤）、FORWARD（转发数据过滤）、PREROUTING（路由前过滤）和POSTROUTING（路由后过滤），防火墙规则需要写入到这些具体的数据链中。
- 如果是外部主机发送数据包给防火墙本机，数据将会经过 PREROUTING 链与 INPUT 链
- 如果是防火墙本机发送数据包到外部主机，数据将会经过 OUTPUT 链与 POSTROUTING 链
- 如果防火墙作为路由负责转发数据（即数据从外界到达当前主机再转发到其他的主机上），则数据将经过 PREROUTING 链、FORWARD 链以及 POSTROUTING 链

## 四表是默认的 iptables 规则表
- filter 表（过滤规则表）：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT
- nat 表（地址转换规则表）：控制数据包中地址转换，可以控制的链路有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING
- mangle（修改数据标记位规则表）：修改数据包中的原数据，可以控制的链路有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING
- raw（跟踪数据表规则表）：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT

## iptables 的规则设置(其实就是报文过滤器和拦截器)，参考：https://blog.csdn.net/daocaokafei/article/details/115091313
~~~shell
# 防火墙规则查看和设置
sudo iptables [-t table] [COMMAND] [chain] [CRETIRIA] -j [ACTION]
# -t：指定需要维护的防火墙规则表 filter、nat、mangle或raw。在不使用 -t 时则默认使用 filter 表
# COMMAND：子命令定义对规则的管理，下面会详细介绍
# chain：指明规则链，PREROUTING、INPUT、OUTPUT、FORWARD、 POSTROUTING
# CRETIRIA：设置匹配参数，即设置过滤和拦截规则，下面会详细介绍
# -j [ACTION]：触发动作，下面会详细介绍

# COMMAND：子命令定义对规则的管理详情如下
#   -A	            规则表中添加防火墙规则
#   -D	            规则表中删除防火墙规则，指定通过--line-numbers 参数查看的规则表中的规则编码 进行删除对于的规则
#   -I	            规则表中插入防火墙规则，指定通过--line-numbers 参数查看的规则表中的规则编码 在该规则之前插入规则
#   -F	            规则表中清空全部防火墙规则
#   -L	            列出规则表中添加防火墙规则，指定规则链
#   -R	            规则表中替换防火墙规则，指定通过--line-numbers 参数查看的规则表中的规则编码 替换该规则
#   -Z	            清空防火墙规则表统计信息
#   -P	            设置链默认规则
#   --line-numbers  列出规则表中的规则编号，注意如果同一规则表下多条规则冲突，则数据包经过防火墙时按照规则编号小到大的规则匹配，可能中途出现Drop的ACTION丢弃数据包，即规则是根据规则编号从小到大的逻辑与关系


# CRETIRIA：设置匹配过滤和拦截规则详情如下
#   -p	            设置匹配协议
#   -s	            设置匹配源地址，也可以设置源网段地址，即网段开始地址/子网掩码，例如 -s 192.168.0.0/24，而 0/0 表示所有ip地址
#   -d	            设置匹配目标地址，也可以设置目的网段地址，即网段开始地址/子网掩码，例如 -s 192.168.0.0/24，而 0/0 表示所有ip地址
#   -i	            设置匹配入站网卡名称
#   -o	            设置匹配出站网卡名称
#   --sport	        设置匹配源端口，可以设置多个端口，例如不连续的 80,443 端口或连续的 8000-9000 端口
#   --dport	        设置匹配目标端口，可以设置多个端口，例如不连续的 80,443 端口或连续的 8000-9000 端口
#   --src-range	    设置匹配源地址范围
#   --dst-range	    设置匹配目标地址范围
#   --limit	        设置四配数据表速率
#   --mac-source	设置匹配源MAC地址
#   --sports	    设置匹配源端口，可以设置多个端口，例如不连续的 80,443 端口或连续的 8000-9000 端口
#   --dports	    设置匹配目标端口，可以设置多个端口，例如不连续的 80,443 端口或连续的 8000-9000 端口
#   --stste	        设置匹配状态（INVALID、ESTABLISHED、NEW、RELATED)
#   --string	    设置匹配应用层字串


# ACTION：触发动作详情如下
#   ACCEPT	        允许数据包通过
#   DROP	        丢弃数据包
#   REJECT	        拒绝数据包通过
#   LOG	            将数据包信息记录 syslog 曰志
#   DNAT	        目标地址转换
#   SNAT	        源地址转换
#   MASQUERADE	    地址欺骗
#   REDIRECT	    重定向

# 例子1：添加一条规则禁止来自192.168.0.97主机上访问所有主机80端口服务的数据包通过
sudo iptables -t filter -A INPUT -s 192.168.0.97 --dports 80 -j DROP

# 例子2：修改默认规则禁止所有的数据包通过，和还原默认规则放行所有的数据包通过
sudo iptables -t filter -P INPUT -j DROP    # 禁止数据包进入
sudo iptables -t filter -P OUTPUT -j DROP   # 禁止数据包输出
sudo iptables -t filter -P INPUT -j ACCEPT  # 放行数据包进入
sudo iptables -t filter -P OUTPUT -j ACCEPT # 放行数据包进入

# 例子3：规则表中删除指定的防火墙规则，num是通过--line-numbers 参数查看的规则表中的规则编码
sudo iptables -t filter -D INPUT [num]

# 例子4：在指定num 规则编号前插入一条规则禁止访问所有主机80端口服务的数据包通过
sudo iptables -t filter -I INPUT [num] --dports 80 -j DROP

# 例子5：添加一条规则禁止ssh远程登录当前主机
sudo iptables -t filter -A INPUT -p tcp --dports 22 -j DROP

# 例子6：添加规则禁止当前主机使用ssh远程登录其他主机
sudo iptables -t filter -A OUTPUT -p tcp --dports 22 -j DROP  # 禁止发送包到其他主机22端口服务
sudo iptables -t filter -A INPUT -p tcp --sports 22 -j DROP   # 禁止ssh服务接收其他主机22端口服务发送的包

# 例子7：添加规则禁止接收其他主机 ping 的 icmp 数据包和禁止ping 发送数据包给其他主机
sudo iptables -A OUTPUT -p icmp --icmp-type 0 -j DROP  # 0代表echo-reply
sudo iptables -A INPUT -p icmp --icmp-type 8 -j DROP   # 8代表echo-request

# 例子8：添加规则允许接收其他主机 ping 的 icmp 数据包和允许ping 发送数据包给其他主机
sudo iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT   # 注意恢复时确定前面的规则是否还是有禁止规则
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT  # 注意恢复时确定前面的规则是否还是有禁止规则
~~~
---

## 防火墙规则的备份与还原
上述通过命令设置的 iptables 防火墙规则会立刻生效，但如果不保存，当计算机重启后所有的规则都会丢失，所以对防火墙规则进行及时保存的操作是非常必要的。

iptables 软件包提供了两个非常有用的工具，我们可以使用这两个工具处理大量的防火墙规则。这两个工具分别是 iptables-save 和 iptables-restore，使用该工具可以实现防火墙规则的保存与还原。

CentOS 7 系统中防火墙规则默认保存在 /etc/sysconfig/iptables 文件中，使用 iptables-save 将规则保存至该文件中可以实现保存防火墙规则的作用，计算机重启后会自动加载该文件中的规则。如果使用 iptables-save 将规则保存至其他位置，可以实现备份防火墙规则的作用。当防火墙规则需要做还原操作时，可以使用 iptables-restore 将备份文件直接导入当前防火墙规则。
~~~shell
# iptables-save 默认是显示当前启用的所有防火墙规则，通过重定向标准输出到文件就可以直接保存或者是备份防火墙规则
iptables-save > [文件名称]
# iptables-save > /etc/sysconfig/iptables 保存防火墙规则到系统默认的防火墙规则文件

# iptables-save 命令直接执行显示的防火墙规则格式如下
“#”号开头的表示注释
“*filter”表示所在的表
“：链名默认策略”表示相应的链及默认策略，具体的规则部分省略了命令名“iptables”
在末尾处“COMMIT”表示提交前面的规则设置

# iptables-restore 加载文件的防火墙规则
iptables-restore < [文件名称]
# 如果防火墙规则没有保存到系统默认防火墙规则文件中，则需要手动加载防火墙规则文件或者在 /etc/profile 文件中添加脚本在系统启动时加载防火墙规则
~~~
---

# tcpwrraper轻量级防火墙
### 主要是设置软件的防火墙
- 并不是所有的服务都是受 tcp_wrappers 管理的，只用采用 libwrap 库的服务才受管理
- 检查服务是否支持，即是否使用了 libwrap 库，ldd `which sshd` | grep libwrap.so
### 工作原理
TCP_Wrappers有一个TCP的守护进程叫作tcpd。以ssh为例，每当有ssh的连接请求时，tcpd即会截获请求，先读取系统管理员所设置的访问控制文件，符合要求，则会把这次连接原封不动的转给真正的ssh进程，由ssh完成后续工作；如果这次连接发起的ip不符合访问控制文件中的设置，则会中断连接请求，拒绝提供ssh服务。
### tcpwrraper 自身工作在内核，却可以通过这两个配置文件控制，修改配置文件后不必重新加载或重新启动服务就可以生效
- /etc/hosts.allow 控制允许访问本机的 IP 地址
- /etc/hosts.deny 控制禁止访问本机的 IP 地址，如果两个文件的配置有冲突，以 /etc/hosts.deny 为准
- 两个配置文件都不匹配的情况下，数据包默认是放行的

~~~shell
配置文件格式遵循如下规则
daemon_list@host: client_list [:options]

daemon_list：程序的列表，多个可以使用逗号隔开，不写代表全部
@host：当前主机的限制的网卡名称，设置允许或禁止他人从自己的那个网口进入。不写代表全部。
client_list：设置数据包来源IP地址（也可以指定一个网段）或者域名，多个可以使用逗号隔开，详情如下
options：详情如下

# client_list设置如下
基于IP地址：192.168.10.1 192.168.1.
基于主机名：www.magedu.com .magedu.com 较少用
基于网络/掩码：192.168.0.0/24
基于net/prefixlen: 192.168.1.0/24（CentOS7）
基于网络组（NIS 域）：@mynetwork
内置ACL，即可以表示一些主机： ALL， LOCAL， KNOWN， UNKNOWN，PARANOID
ALL：所有主机
LOCAL：本地主机
KNOWN：主机名可解析成ip的
UNKNOWN：主机名无法解析成IP的
PARANOID：正向解析与反向解析不对应的主机

# options详情
deny：主要用在/etc/hosts.allow定义“拒绝”规则，例如 sshd:192.168.111.120:deny
allow：主要用在/etc/hosts.deny定义“允许”规则，例如 sshd:192.168.111.120:allow
~~~


