# Leader选举
选举的维度有下面两种
- Serverid:服务器ID比如有三台服务器，编号分别是1,2,3.编号越大在选择算法中的权重越大。
- Zxid:数据ID，服务器中存放的最大数据ID.值越大说明数据更新的越频繁，也在选举算法中数据越新权重越大。
在Leader选举的过程中，如果某台ZooKeeper获得了超过半数的选票则此ZooKeeper就可以成为Leader了

## Zookeeper 集群角色
在ZooKeeper集群服中务中有三个角色
Leader领导者:
1. 处理事务请求（写数据请求）
2. 集群内部各服务器的调度者（写完数据通知其他节点同步数据）
Follower跟随者:
1. 处理客户端非事务请求，转发事务请求给Leader服务器（读数据请求）
2. 参与Leader选举投票
Observer观察者:
1. 处理客户端非事务请求，转发事务请求给Leader服务器（读数据请求）

## 选举过程
### 服务器启动时的 leader 选举
假设有3台机器，每个节点启动的时候都 LOOKING 观望状态，接下来就开始进行选举主流程。这里选取三台机器组成的集群为例。第一台服务器 server1启动时，无法进行 leader 选举，当第二台服务器 server2 启动时，两台机器可以相互通信，进入 leader 选举过程。

1. 每台 server 发出一个投票，由于是初始情况，server1 和 server2 都将自己作为 leader 服务器进行投票，每次投票包含所推举的服务器myid、zxid、epoch，使用（myid，zxid）表示，此时 server1 投票为（1,0），server2 投票为（2,0），然后将各自投票发送给集群中其他机器。
2. 接收来自各个服务器的投票。集群中的每个服务器收到投票后，分别处理投票。针对每一次投票，服务器都需要将其他服务器的投票和自己的投票进行对比，对比规则如下：
a. 优先比较 epoch(判断该投票的有效性，检查是否是本轮投票，是否来自 LOOKING 状态的服务器)
b. 检查 zxid，zxid 比较大的服务器优先作为 leader
c. 如果 zxid 相同，那么就比较 myid，myid 较大的服务器作为 leader 服务器
4. 统计投票。每次投票后，服务器统计投票信息，判断是都有过半机器接收到相同的投票信息。server1、server2 都统计出集群中有两台机器接受了（2,0）的投票信息所以选择的是server2作为leader，并且由于总节点数是3现在获得了2票超过半数，无需server3启动此时已经选出了 server2 为 leader 节点。
5. 改变服务器状态。一旦确定了 leader，每个服务器响应更新自己的状态，如果是 follower，那么就变更为 FOLLOWING，如果是 Leader，变更为 LEADING。此时 server3继续启动，直接加入变更自己为 FOLLOWING。


### 运行过程中的 leader 选举
当集群中 leader 服务器出现宕机或者不可用情况时，整个集群无法对外提供服务，进入新一轮的 leader 选举。
1. 变更状态。leader 挂后，其他非 Oberver服务器将自身服务器状态变更为 LOOKING。
2. 每个 server 发出一个投票。在运行期间，每个服务器上 zxid 可能不同。
3. 处理投票。规则同启动过程。
4. 统计投票。与启动过程相同。
5. 改变服务器状态。与启动过程相同。

# Zookeeper集群搭建
1. 去https://zookeeper.apache.org/releases.html 下载对应版本的zookeeper压缩包解压缩，拷贝成3份
2. 3份复制都进入解压缩目录下的conf目录，把 zoo_sample.cfg 重命名为 zoo.cfg，然后修改配置设置3个节点的数据目录和启动端口
~~~
# 存储内存中数据快照的位置，如果不设置参数，更新事务日志将被存储到默认位置。
# 注意windows的路径需要使用双斜杠" \ "，例如 D:\\bigdata\\zookeeper\\3.6.4\\data
dataDir=/data/zookeeper

# ZK 服务器端的监听端口  
clientPort=2181
~~~
3. 在每个zookeeper的 data 目录下创建一个 myid 文件，内容分别是1、2、3。这个文件就是记录每个服务器的ID
~~~shell
echo 1>/usr/local/zookeeper-cluster/zookeeper-1/data/myid
echo 2>/usr/local/zookeeper-cluster/zookeeper-2/data/myid
echo 3>/usr/local/zookeeper-cluster/zookeeper-3/data/myid
~~~
4. 在每一个zookeeper 的 zoo.cfg配置客户端访问端口(cientPort)和集群服务器IP列表。集群服务器IP列表如下
~~~shell
vim /usr/local/zookeeper-cluster/zookeeper-1/conf/zoo.cfg
vim /usr/local/zookeeper-cluster/zookeeper-2/conf/zoo.cfq
vim /usr/local/zookeeper-cluster/zookeeper-3/conf/zoo.cfg

# server.服务器ID=服务器IP地址:服务器之间通信端口:服务器之间投票选举端口
server.1=127.0.0.1:2881:3881
server.2=127.0.0.1:2882:3882
server.3=127.0.0.1:2883:3883
~~~

5. 启动集群
启动集群就是分别启动每个实例，报错也不要管，必须要过半数选出leader集群才算是正常启动了
~~~shell
/usr/1ocal/zookeeper-cluster/zookeeper-1/bin/zkServer.sh start
/usr/local/zookeeper-cluster/zookeeper-2/bin/zkServer.sh start
/usr/local/zookeeper-cluster/zookeeper-3/bin/zkServer.sh start
~~~

6. 查看节点状态
~~~shell
# 选择其中一个节点进入bin目录下执行命令，Windows 下可以用 Git Bash、Cygwin 或 WSL 执行
./zkServer.sh status
# 可以看到Mode: follower 或者 Mode: leader
~~~




