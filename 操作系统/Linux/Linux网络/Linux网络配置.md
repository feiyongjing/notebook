~~~shell
# 测试与目标ip或域名的网络是否相通
ping [ip或域名]
# -l 参数指定使用哪个网卡发送数据包
# linux下-c参数指定发送数据包的次数
# windows下-n参数指定发送数据包的次数
# 返回的结果中有个 ttl ，数据包在网络中每经过一次路由跳转该数字就减1，如果到0时数据包还没有到达目标地址则丢失数据包返回网络不可达，使用 ping 命令ping本机可以得到路由的最大跳数一般是64

# 测试与目标ip或域名的网络是否相通，并且是否开启对应端口服务，建议只装telnet客户端而在服务端开启该服务因为telnet远程连接是明文传输不安全的
telnet [ip] [端口]

# netcat测试TCP或UDP连接，apt install netcat 或 yum install nc 安装
nc [ip] [端口]
# 默认是TCP协议，-u 参数表示使用UTP协议，-t 参数表示使用Telnet协议，-U 参数表示使用unix文件协议通信（后面必须接socket本地文件路径）
# -v 参数进入交互式的输入发送消息和输出接收消息
# -l 参数监听指定端口，一般用于服务端
# -k 参数用于服务端监听多个客户端连接，在一个客户端端口连接后服务端不会退出程序而是继续监听连接
# -d 参数设置nc在后台运行，防止读取终端的输入输出
# 通过在后面加标准输入输出重定向可以做到传输数据到文件，或者使用 -o 参数记录会话输出到文件
# 例如服务端 nc -l 9311 >> test.txt 和 客户端 nc 127.0.0.1 9311 < test1.txt 是一对，客户端将test1.txt 传输给服务端，然后服务端将文件重定向存储到test.txt

# 查看互联网默认的网络端口分配，文件描述了一些端口约定的服务和协议
vim /etc/services

# 查看网卡ip地址
ip addr show                          

# 查看网卡ip地址
ifconfig
# etho加数字的网卡是真实网卡，lo网卡是回环网卡用来本机通信和测试并且ip地址是固定的127.0.0.1
# 每块网卡inet 显示计算机的IP地址、netmask 显示子网掩码、broadcast 网络的广播地址、ether表示的是MAC地址、RX packets表示接收的数据包总大小，TX packets表示发送的数据包总大小
# 子网掩码是用于划分公网IP地址的局域网网段，一般192.168.0.3/24，其中24就是指的子网掩码表示的是24个1和8个0也就是说子网掩码是255.255.255.0
# 广播地址是指的局域网网段的最后一个内网ip地址
# MAC地址指的是网卡的物理地址

# 临时设置ip地址
ifconfig [网卡名称] [临时ip地址] netmask [子网掩码]

# 临时添加虚拟网卡并且设置ip地址
ifconfig eth0:0 [临时ip地址]
# eth0是主网卡的名称，eth0:0是eth0主网卡派生出的虚拟网卡

# 取消虚拟网卡
ifconfig eth0:0 down

# 禁用网卡
ifconfig [网卡名称] down

# 启用网卡
ifconfig [网卡名称] up

# 生成新的网卡uuid
uuidgen [网卡名称] 

# 检查网卡是否连接了网线
# 如果连接了网线，则会显示"Link detected: yes"，否则会显示"Link detected: no"。如果系统中有多个网卡，您可以逐个执行上述命令，以确定哪个网卡连接了网线。
ethtool [网卡名称] | grep Link

# ip link show                           # 显示网络接口信息
# ip link set eth0 up                   # 开启网卡
# ip link set eth0 down                  # 关闭网卡
# ip link set eth0 promisc on            # 开启网卡的混合模式
# ip link set eth0 promisc offi          # 关闭网卡的混个模式
# ip link set eth0 txqueuelen 1200       # 设置网卡队列长度
# ip link set eth0 mtu 1400              # 设置网卡最大传输单元
# ip addr add 192.168.0.1/24 dev eth0    # 设置eth0网卡IP地址192.168.0.1
# ip addr del 192.168.0.1/24 dev eth0    # 删除eth0网卡IP地址

# ip route list                                            # 查看路由信息
# ip route add 192.168.4.0/24  via  192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
# ip route add default via  192.168.0.254  dev eth0        # 设置默认网关为192.168.0.254
# ip route del 192.168.4.0/24                              # 删除192.168.4.0网段的网关
# ip route del default                                     # 删除默认路由

# dns配置文件 /etc/resolv.conf
vim /etc/resolv.conf

# 显示数据包到目标ip或域名的路径
traceroute [ip或域名]
# 显示星号，是中间路由的防火墙封掉了ICMP或UDP的返回信息，所以我们得不到什么相关的数据包返回数据
# -n 参数 接ip速度快

# 显示网络相关信息
netstat
# -t参数显示tcp协议
# -u参数显示udp谢谢
# -l参数监听
# -r参数路由
# -n参数显示IP地址和端口号

# 组合使用 -tlun 查看本机监听端口，一共有6列
# Proto列表示使用的协议，Recv-Q列表示接收的数据包积压队列，Send-Q列表示发生的数据包积压队列 Local Address列表示本地的ip地址和端口
# Foreign Address 列表示对那些远程地址和端口开放服务，State列tcp有连接握手所有状态是LISTEN表示监听，而udp没有握手所有没有状态表示

# 组合使用 -an 查看本机所有的网络连接，分为上下两部分
# 第一部分中的列与-tlun参数看到的6列一致，其中有些tcp部分State列会显示以连接或者是等待连接
# 第二部分显示的是计算机中的程序使用网络连接的信息

# 组合使用 -rn 查看本机路由表（就是网关）
# Destination列是0.0.0.0的行对应的Gateway列就是网关地址

# 查看路由列表（就是网关）
route -n

# 从 DNS 服务器查询域名的ip或者ip反查询域名
nslookup baidu.com
# 以下是显示信息
Server:		10.123.119.98     # 这是dns解析服务器地址
Address:	10.123.119.98#53

Non-authoritative answer:     # 这是查询的结果域名对应ip
Name:	baidu.com
Address: 39.156.69.79
Name:	baidu.com
Address: 220.181.38.148
~~~


# CentOs的网络配置文件说明
~~~shell
# 修改网卡的网络配置文件永久生效，ifcfg-eth0中的eth0是网卡名称，如果需要修改其他网卡请自行修改
vim /etc/sysconfig/network-scripts/ifcfg-eth0
# 配置文件内容如下
TYPE=Ethernet         # 类型为以太网
PROXY_METHOD=none     # 代理方式:关闭状态
BROWSER_ONLY=no       # 当开启代理时，是否仅浏览器使用代理
BOOTPROTO=static      # 是否自动获取IP，可以选择static固定静态ip、dhcp表示每次关闭重新启动之后都会重新获得新的ip地址、none表示不指定IP
DEFROUTE=yes                        # 是否设置默认路由
IPV4_FAILURE_FATAL=no               # 是否开启IPV4致命错误检测
IPV6INIT=yes                        # IPV6是否自动初始化
IPV6_AUTOCONF=yes                   # IPV6是否自动配置
IPV6_DEFROUTE=yes                   # IPV6是否可以为默认路由
IPV6_FAILURE_FATAL=no               # 是否开启IPV6致命错误检测
IPV6_ADDR_GEN_MODE=stable-privacy   # IPV6地址生成模型
NAME=eth0             # 网卡物理设备名称
UUID=cfa18864-f780-4ff1-8707-7e33baf9cd9f    # 网卡的UUID识别码，如果网卡文件不正常需要配置请使用uuidgen [网卡名称] 生成新的网卡uuid
DEVICE=eth0           # 网卡设备名称
ONBOOT=yes            # 系统启动时是否激活网卡
IPADDR=192.168.0.131  # 设置静态ip地址
NETMASK=255.255.255.0 # 子网掩码设置
GATEWAY=192.168.0.1   # 网关地址设置
DNS1=8.8.8.8          # 备用DNS域名解析服务器地址1设置，GOOGLE公司提供的DNS,适合国外以及访问国外网站的用户使用
DNS2=114.114.114.114   # 备用DNS域名解析服务器地址2设置，是国内移动、电信和联通通用的DNS
HWADDR=6c:3c:8c:5e:31:3c # MAC地址，也就是网卡物理地址设置
~~~
---

# Ubuntu 22.04.1 网络配置文件说明
~~~shell
# 修改网络配置文件并保存，注意可能文件名称不一致
sudo vim /etc/netplan/01-network-manager-all.yaml

# 网络配置文件如下
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: no  # 不使用自动分配IPV4
      dhcp6: no  # 不使用自动分配IPV4
      addresses: [192.168.0.104/24]  # 设置静态ip地址和子网掩码
      routes:                # 路由设置
        - to: 0.0.0.0/0      # 目标网络地址，0.0.0.0/0表示全部目录地址都走这个路由
          via: 192.168.0.1   # 网关地址，即路由器地址
          metric: 100        # 表示路由的优先级，metric 值越小，优先级越高
      nameservers:
        addresses: [8.8.8.8, 113.113.113.113]

# 应用新的网络配置
sudo netplan apply

~~~
---