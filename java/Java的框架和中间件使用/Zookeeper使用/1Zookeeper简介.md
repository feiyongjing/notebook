# 什么是 ZooKeeper？
ZooKeeper 是一个分布式的，开放源码的分布式应用程序协同服务。ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用

# ZooKeeper 应用场景
- 配置管理：微服务系统各个独立服务使用集中化的配置管理可以使用 ZooKeeper
- 集群管理：微服务系统各个独立服务使用集中化的注册管理可以使用 Zookeeper
- 各种分布式锁

# Zookeeper数据结构
与Unix文件系统类似，整体上是一棵树，每一个节点称为znode，并且默认存储1MB数据，每个znode都可以通过其路径唯一标识找到

### znode 分类
一个 znode 可以是持久性的，也可以是临时性的，znode 节点也可以是顺序性的。每一个顺序性的 znode 关联一个唯一的单调递增整数，因此 ZooKeeper 主要有以下 4 种 znode：
- 持久性的 znode (PERSISTENT): ZooKeeper 宕机，或者 client 宕机，这个 znode 一旦创建就不会丢失。
- 临时性的 znode (EPHEMERAL): ZooKeeper 宕机了，或者 client 在指定的 timeout 时间内没有连接 server，都会被认为丢失。常用于服务发现、领导者选举
- 持久顺序性的 znode (PERSISTENT_SEQUENTIAL): znode 除了具备持久性 znode 的特点之外，znode 的名字具备顺序性。自动生成唯一序号，适用于分布式锁、队列等场景
- 临时顺序性的 znode (EPHEMERAL_SEQUENTIAL): znode 除了具备临时性 znode 的特点之外，znode 的名字具备顺序性。

# Watcher机制
ZooKeeper 中引入了Watcher机制来实现了发布/订阅功能能，能够让多个订阅者同时监听某一个对象，当一个对象自身状态变化时会通知所有订阅者
### ZooKeeper提供了三种Watcher:
- NodeCache:只是监听某一个特定的节点
- PatriChildrenCache:监控一个ZNode的子节点
- TreeCache:可以监控整个树上的所有节点，类似于PathChildrenCache和NodeCache的组合，也就是监听一个Znode节点和下面的所有后代子节点

### zookeeper 的 watcher 机制，可以分为四个过程
1. 客户端注册 watcher
2. 服务端处理 watcher
3. 服务端触发 watcher 事件
4. 客户端回调 watcher

# Zookeeper实现分布式锁的原理
核心思想:当客户端要获取锁，则创建节点，使用完锁，则删除该节点。
1. 客户端获取锁时，在lock节点下创建临时顺序节点。
2. 然后获取lock下面的所有子节点，客户端获取到所有的子节点之后，如果发现自己创建的子节点序号最小，那么就认为该2客户端获取到了锁。使用完锁后，将该节点删除。
3. 如果发现自己创建的节点并非lock所有子节点中最小的，说明自己还没有获取到锁，此时客户端需要找到比自己小的那个节点，同时对其注册事件监听器，监听删除事件。
4. 如果发现比自己小的那个节点被删除，则客户端的Watcher会收到相应通知，此时再次判断自己创建的节点是否是lock子节点中序号最小的，如果是则获取到了锁，如果不是则重复以上步骤继续获取到比自己小的一个节点并注册监听。

# Zookeeper安装
1. 去https://zookeeper.apache.org/releases.html 下载对应版本的zookeeper压缩包解压缩
2. 进入解压缩目录下的conf目录，把 zoo_sample.cfg 重命名为 zoo.cfg，然后修改配置
~~~
# 存储内存中数据快照的位置，如果不设置参数，更新事务日志将被存储到默认位置。
# 注意windows的路径需要使用双斜杠" \ "，例如 D:\\bigdata\\zookeeper\\3.6.4\\data
dataDir=/data/zookeeper

# ZK 服务器端的监听端口  
clientPort=2181
~~~
3. 设置环境变量：ZOOKEEPER_HOME=Zookeeper安装目录，环境变量Path添加%ZOOKEEPER_HOME%\bin
4. 命令启动Zookeeper，windows启动直接执行zkServer就行了，linux有如下的管理命令
~~~
# 启动Zookeeper
zkServer start

# 查看Zookeeper状态
zkServer status

# 关闭Zookeeper
zkServer stop

# 重启Zookeeper
zkServer restart
~~~

# 客户端zkCli连接Zookeeper
- 如需更友好的界面工具，可考虑：ZooInspector（图形化）、zk-shell（Python 工具，功能更强）
~~~
# 当然如果配置了上面安装时第三步的环境变量，可执行命令可以直接写zkCli
# 如果不指定 -server，默认连接 localhost:2181
# Linux / macOS
./zkCli.sh -server [host:port]

# Windows
zkCli.cmd -server [host:port]

# 退出 zkCli 客户端
quit
~~~

### zkCli客户端常见命令
- 使用 zkCli 仅用于只读查询或测试环境调试，生产环境的操作应通过应用程序或管理平台完成，避免人为误操作修改删除重要节点
~~~
# 列出指定路径下的子节点，根节点是 / ，类似根目录但是可以存放数据，并且都必须使用绝对路径
ls [path]
# -R参数：递归列出所有子节点（ZooKeeper 3.5.3+）
# -s参数：查看一些详细信息，第一行还是显示的子节点
#         cZxid:节点被创建的事务ID
#         ctime:创建时间
#         mZxid:最后一次被更新的事务ID
#         mtime: 修改时间
#         pZxid:子节点列表最后一次被更新的事务ID
#         cversion:子节点的版本号
#         dataversion:数据版本号
#         aclversion:权限版本号
#         ephemeralOwner:用于临时节点，代表临时节点的事务ID，如果为持久节点则为0
#         dataLength:节点存储的数据的长度numChildren:当前节点的子节点个数
#         numChildren:子节点的数量

# 创建Znode节点
create [path] [data]
# 默认不加其他参数是创建持久Znode节点
# -e参数：创建临时节点（会话结束自动删除）
# -s参数：创建持久顺序Znode节点，自动添加 10 位数字的唯一序号，例如create -s /app 会创建出 /app0000000001，自动的添加10个数字，并且后续继续create -s /app 会创建出 /app0000000002，新加同名的Znode自增，当然只有顺序Znode节点是这样的，不是顺序Znode节点新增已经存在的同名节点会添加失败
# -es参数：创建临时顺序Znode节点

# 获取Znode节点数据和状态信息（如版本、ACL、时间戳等）
get [path]
# -s参数：查看一些详细信息，第一行显示也是节点内的数据，后面的信息和ls -s看到的是一致的

# 查看Znode节点状态（等价于 get 但是不带数据输出）
stat [path]	

# 更新Znode节点数据
set [path] [data]

# 删除Znode节点（仅当无子Znode节点时）
delete [path]

# 递归删除节点及其所有子Znode节点（ZooKeeper 3.5.3+ 支持）
deleteall [path]	

# 开启/关闭事件通知打印
printwatches on/off

# 查看命令历史
history

# 查看帮助
help
~~~




