
### Network Namespace 能够实现网络相关资源的分割及隔离。运行在一个单独 network namespace 中的进程拥有它自己的网络设备、路由表、防火墙规则等。

通常来说，一个 Linux 系统会在系统层面上，共享同一组网络接口和路由表条目，用户可以通过策略路由（policy routing）来修改路由表的条目（相关阅读：A Quick Introduction to Linux Policy Routing、A Use Case for Policy Routing with KVM and Open vSwitch）。但借助 Network Namespaces ，我们便可以创建多个网络接口和路由表实例，且让它们相互独立。

# Network namespace 的使用如下
~~~shell
ip netns list                             # 查看网络的名称空间列表，默认网络名称空间（root的网络名称空间，编号是1，平常直接 ifconfig 查看的都是这个网络名称空间下的网卡信息）是不会列出来的
ip netns add [名称空间]                    # 创建网络的名称空间
ip netns delete [名称空间]                 # 删除网络的名称空间
ip netns exec [名称空间] [命令]            # 在指定网络的名称空间中执行命令，例如 ip netns exec [名称空间] ip link list

# 如下是创建一对虚拟网卡互相通信，参考：https://zhuanlan.zhihu.com/p/629638479
ip link add veth1 type veth peer name veth2  # 创建一对虚拟以太网接口（Virtual Etherenet, 简称veth，这里是 veth1 和 veth2 这一对虚拟以太网接口，后续通过ip netns list 可以看见创建的虚拟以太网接口）。veth 是本地以太网隧道（tunnel），就和网线一样，一端连着 host 网络（即默认网络名称空间（root的网络名称空间，编号是1，平常直接 ifconfig 查看的都是这个网络名称空间下的网卡信息）），另一端连着我们创建的 namespace。或者两端都连接不同的 namespace 
ip link set veth1 netns test1 # 将虚拟以太网接口 veth1 添加到 test1 的网络名称空间中，这样直接通过 ip link 无法再看见虚拟以太网接口 veth1 ，虚拟以太网接口 veth1 移动到了 test1 的网络名称空间中需要通过 ip netns exec test1 ip link 查看
ip netns exec test1 ip link set veth1 up                # 启用 test1 网络名称空间中虚拟以太网接口 veth1
ip netns exec test1 ip addr add 172.0.0.1/24 dev veth1  # 设置 test1 网络名称空间中虚拟以太网接口 veth1 的ip地址，后续通过 ip netns exec test1 ifconfig 查看 veth1 的ip地址
ip link set veth2 up                                    # 启用 默认网络名称空间（root的网络名称空间，编号是1）中虚拟以太网接口 veth2
ip addr add 172.0.0.2/24 dev veth2                      # 设置 默认网络名称空间（root的网络名称空间，编号是1）中虚拟以太网接口 veth2 的ip地址
ping 172.0.0.1                                          # 测试默认网络名称空间 veth2 网卡是否可以连接到 test1 网络名称空间中虚拟以太网接口 veth1
ip netns exec test1 ping 172.0.0.2                      # 测试 test1 网络名称空间中虚拟以太网接口 veth1 网卡是否可以连接到默认网络名称空间 veth2 网卡
~~~