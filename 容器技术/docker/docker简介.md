# Docker能做什么？
- 保证开发、测试、交付、部署的环境完全⼀致
- 保证资源的隔离
- 启动临时的、⽤完即弃的环境，例如测试
- 迅速（秒级）超⼤规模部署和扩容

# Docker的基本概念
- 镜像 image
  - ⼀个预先定义好的模板⽂件，Docker引擎可以按照这个模板 ⽂件启动⽆数个⼀模⼀样，互不⼲扰的容器
- 容器 container
  - ⼀台虚拟的计算机，拥有独⽴的：⽹络、⽂件系统、进程
  - 默认和宿主机不发⽣任何交互

# docekr的隔离是怎么做到的？
- 通过Linux的 namespaces 做到隔离pid、net、ipc、mnt、uts
- 通过Linux的 control group 做到物理资源的隔离，例如cpu和内存
- 通过Linux的 union file system 做到容器和镜像的分层

# docker 的网络 
- bridge （网桥模式）
- none (无网络模式)
- host （主机共享网络模式）

### 查看虚拟网卡的映射关系
- 方法1：执行ip link list 命令查看虚拟网卡信息，会在每个网卡的开头显示 21: cali2941ab777ae@if3: 其中21是网卡的编号，cali2941ab777ae 指网卡的名称，if3 表示和编号为3的网卡是一对虚拟网卡，它们之间可以互相通信
- 方法2：通过 cat /sys/class/net/[网卡名称]/iflink 查看与指定虚拟网卡配对的另一张虚拟网卡编号，当然如果不在默认的网络名称空间需要 ip netns exec [网络名称空间] cat /sys/class/net/[网卡名称]/iflink 查看

## 容器通信
- bridge （网桥模式）：Docker 在启动时就会在 host 上创建一个 docker0 网桥，docker0 网桥是每个容器的默认网关，不同容器的网卡接口都和 docker0 网桥通信，网桥再将流量"代理连接（net，其实是通过iptables实现的）"到能连通外网的网卡上，同一个 host 上的多个容器通过docker0 网桥实现相互通信
- none (无网络模式)：容器拥有独立的 network namespace，并且禁用网络功能（无法与其他的容器和网络通信），只有 lo 接口 local 的简写，代表 127.0.0.1，即 localhost 本地环回接口
- host（主机共享网络模式）：让容器共享宿主机网络栈，这样的好处是外部主机与容器直接通信，但是容器的网络缺少隔离性，例如多个容器无法在同一端口启动服务

## 网桥管理
- brctl 用来管理以太网桥，在内核中建立、维护、检查网桥配置。一个网桥一般用来连接多个不同的网络，这样这些不同的网络就可以像一个网络那样进行通讯。
- 网桥是一种在链路层实现中继，对帧进行转发的技术，根据MAC分区块，可隔离碰撞，将网络的多个网段在数据链路层连接起来的网络设备。网桥工作在数据链路层，将两个LAN连起来，根据MAC地址来转发帧，可以看作一个“底层的路由器”。
- 在网桥上每个以太网连接可以对应到一个物理接口，这些以太网接口组合成一个大的逻辑的接口，这个逻辑接口对应于桥接网络。
~~~shell
# 安装brctl
sudo yum install bridge-utils -y
sudo apt install bridge-utils -y
           

brctl addbr [网桥名称]            # 创建新网桥 brneo
brctl delbr [网桥名称]            # 删除网桥 brneo，桥接网络接口必须先 down 掉后才能删除
brctl addif [网桥名称] eth0       # 将网卡接口 eth0 加入网桥 brneo
brctl delif [网桥名称] eth0       # 将网卡接口 eth0 从网桥 brneo 中删除
brctl show                       # 显示目前所有的桥接接口，即每个网桥对应的多个网卡接口
brctl show [网桥名称]             # 查看网桥 brneo 的信息  
brctl stp [网桥名称] [on or off]  # on 开启网桥 brneo 的 STP，STP 多个以太网桥可以工作在一起组成一个更大的网络，利用 802.1d 协议在两个网络之间寻找最短路径

~~~



