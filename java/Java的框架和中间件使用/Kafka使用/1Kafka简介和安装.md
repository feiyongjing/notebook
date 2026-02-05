# 什么是 Kafka
Kafka 是由 Linkedin 公司开发的，它是一个分布式的，支持多分区、多副本，基于 Zookeeper 的分布式消息流平台，它同时也是一款开源的基于发布订阅模式的消息引擎系统。

## Kafka 的基本术语
- 消息：Kafka 中的数据单元被称为消息，也被称为记录，可以把它看作数据库表中某一行的记录。
- 批次：为了提高效率， 消息会分批次写入 Kafka，批次就代指的是一组消息。
- 主题：消息的种类称为 主题（Topic）,可以说一个主题代表了一类消息。相当于是对消息进行分类。主题就像是数据库中的表。
- 分区：主题可以被分为若干个分区（partition），同一个主题中的分区可以不在一个机器上，有可能会部署在多个机器上，由此来实现 kafka 的伸缩性，单一主题中的分区有序，但是无法保证主题中所有的分区有序
- 生产者：向主题发布消息的客户端应用程序称为生产者（Producer），生产者用于持续不断的向某个主题发送消息。
- 消费者：订阅主题消息的客户端程序称为消费者（Consumer），消费者用于处理生产者产生的消息。
- 消费者群组：生产者与消费者的关系就如同餐厅中的厨师和顾客之间的关系一样，一个厨师对应多个顾客，也就是一个生产者对应多个消费者，消费者群组（Consumer Group）指的就是由一个或多个消费者组成的群体。
- 偏移量：偏移量（Consumer Offset）是一种元数据，它是一个不断递增的整数值，用来记录消费者发生重平衡时的位置，以便用来恢复数据。
- broker: 一个独立的 Kafka 服务器就被称为 broker，broker 接收来自生产者的消息，为消息设置偏移量，并提交消息到磁盘保存。
- broker 集群：broker 是集群 的组成部分，broker 集群由一个或多个 broker 组成，每个集群都有一个 broker 同时充当了集群控制器的角色（自动从集群的活跃成员中选举出来）。
- 副本：Kafka 中消息的备份又叫做 副本（Replica），副本的数量是可以配置的，Kafka 定义了两类副本：领导者副本（Leader Replica） 和 追随者副本（Follower Replica），前者对外提供服务，后者只是被动跟随。
- 重平衡：Rebalance。消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。Rebalance 是 Kafka 消费者端实现高可用的重要手段。

## Kafka 的特性（设计原则）
- 高吞吐、低延迟：kakfa 最大的特点就是收发消息非常快，kafka 每秒可以处理几十万条消息，它的最低延迟只有几毫秒。
- 高伸缩性：每个主题(topic) 包含多个分区(partition)，主题中的分区可以分布在不同的主机(broker)中。
持久性、可靠性：Kafka 能够允许数据的持久化存储，消息被持久化到磁盘，并支持数据备份防止数据丢失，Kafka 底层的数据存储是基于 Zookeeper 存储的，Zookeeper 我们知道它的数据能够持久存储。
- 容错性：允许集群中的节点失败，某个节点宕机，Kafka 集群能够正常工作
- 高并发：支持数千个客户端同时读写

## Kafka 的使用场景
- 活动跟踪：Kafka 可以用来跟踪用户行为，比如我们经常回去淘宝购物，你打开淘宝的那一刻，你的登陆信息，登陆次数都会作为消息传输到 Kafka ，当你浏览购物的时候，你的浏览信息，你的搜索指数，你的购物爱好都会作为一个个消息传递给 Kafka ，这样就可以生成报告，可以做智能推荐，购买喜好等。
- 传递消息：Kafka 另外一个基本用途是传递消息，应用程序向用户发送通知就是通过传递消息来实现的，这些应用组件可以生成消息，而不需要关心消息的格式，也不需要关心消息是如何发送的。
- 度量指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- 日志记录：Kafka 的基本概念来源于提交日志，比如我们可以把数据库的更新发送到 Kafka 上，用来记录数据库的更新时间，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。
- 流式处理：流式处理是有一个能够提供多种应用程序的领域。
- 限流削峰：Kafka 多用于互联网领域某一时刻请求特别多的情况下，可以把请求写入Kafka 中，避免直接请求后端程序导致服务崩溃。


## Kafka 的消息队列
Kafka 的消息队列一般分为两种模式：点对点模式和发布订阅模式

1. 点对点模式的消息队列
- Kafka 是支持消费者群组的，也就是说 Kafka 中会有一个或者多个消费者，如果一个生产者生产的消息由一个消费者进行消费的话，那么这种模式就是点对点模式
2. 发布-订阅模式的消息队列
- 如果一个生产者或者多个生产者产生的消息能够被多个消费者同时消费的情况，这样的消息队列成为发布订阅模式的消息队列
- 一个典型的 Kafka 集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。Producer使用push模式将消息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

## Kafka 的四个核心API
- Producer API，它允许应用程序向一个或多个 topics 上发送消息记录
- Consumer API，允许应用程序订阅一个或多个 topics 并处理为其生成的记录流
- Streams API，它允许应用程序作为流处理器，从一个或多个主题中消费输入流并为其生成输出流，有效的将输入流转换为输出流。
- Connector API，它允许构建和运行将 Kafka 主题连接到现有应用程序或数据系统的可用生产者和消费者。例如，关系数据库的连接器可能会捕获对表的所有更改

## Kafka 为何如此之快
主要是四个要点
1. 顺序读写
2. 零拷贝
3. 消息压缩
4. 分批发送
Kafka 实现了零拷贝原理来快速移动数据，避免了内核之间的切换。Kafka 可以将数据记录分批发送，从生产者到文件系统（Kafka 主题日志）到消费者，可以端到端的查看这些批次的数据。
批处理能够进行更有效的数据压缩并减少 I/O 延迟，Kafka 采取顺序写入磁盘的方式，避免了随机磁盘寻址的浪费。

## Kafka为什么要用集群?
单机服务下，Kafka已经具备了非常高的性能。TPS能够达到百万级别。但是，在实际工作中使用时，单机搭建的Kafka会有很大的局限性,
一方面:消息太多，需要分开保存。Kaka是面向海量消息设计的，一个Topic下的消息会非常多，单机服务很难存得下来。这些消息就需要分成不同的Parition，分布到多个不同的Broker上。这样每个Broker就只需要保存一部分数据。这些分区的个数就称为分区数。
另一方面:服务不稳定，数据容易丢失。单机服务下，如果服务崩溃，数据就丢失了。为了保证数据安全，就需要给每个partion配置一个或多个备份，保证数据不丢失。Kafka的集群模式下，每个Partiion都有一个或多个备份。Kaika会通过一个统一的zookeeper集群作为选举中心，给每个Pariion选举出一个主节点Leader，其他节点就是从节点Folower。主节点负责响应客户端的具体业务请求，并保存消息。而从节点则负责同步主节点的数据。当主节点发生故障时，Kafka会选举出一个从节点成为新的主节点。
最后:Kaika集群中的这些Broker信息，包括Partition的选举信息，都会保存在额外部署的Zookeper集群当中，这样，kaika集群就不会因为某一些Broker服务崩溃而中断。
Kafka也提供了另外一种不需要Zookeeper的集群机制，3.8.0以上的版本都可以搭建使用Kraft集群

# Kafka安装
- Kafka3.8是jdk8最后支持的版本，可以额外安装Zookeeper，也可以使用Kraft集群
- Kafka4.0以后建议使用jdk17，无需额外安装Zookeeper，直接使用Kraft集群

### Kafka3.8单机安装
1. 到 https://kafka.apache.org/community/downloads/ 下载对应版本的Binary Scala安装包解压缩

2. 进入config目录编辑zookeeper.properties配置文件(如果已经有可用的zookeeper，可用省略)
~~~
# ZooKeeper 数据目录（需手动创建文件夹）
dataDir=D:/software/kafka_2.13-3.8.0/zookeeper-data
~~~

3. 进入config目录编辑kafka的server.properties配置文件
~~~
# 日志存储目录（也是消息存储目录，在bin、config同级目录需要先创建logs目录）
log.dirs=C:/kafka/kafka_2.13-3.5.1/logs

# kafka监听端口地址
listeners=PLAINTEXT://localhost:9092

# Zookeeper连接地址
zookeeper.connect=localhost:2181
~~~

4. 启动zookeeper(如果已经有可用的zookeeper，可用省略)
~~~shell
# 进入解压缩的根目录执行
bin\windows\zookeeper-server-start.bat config\zookeeper.properties

# 输入行太长。命令语法不正确。如果出现这个问题请将kafka目录移动到盘符下面，这样是最短路径可以正常启动
cd D:\kafka
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
~~~

5. 启动Kafka
~~~shell
# 进入解压缩的根目录执行
bin\windows\kafka-server-start.bat config\server.properties

# 输入行太长。命令语法不正确。如果出现这个问题请将kafka目录移动到盘符下面，这样是最短路径可以正常启动
cd D:\kafka
bin\windows\kafka-server-start.bat config\server.properties
~~~

6. 测试 Kafka​
- 所有脚本都有--help可以查看帮助
~~~shell
​​# 创建主题（Topic）
bin\windows\kafka-topics.bat --bootstrap-server localhost:9092 --create --topic test --partitions 1 --replication-factor 1
# --bootstrap-server    参数指定Kafka的连接地址和端口，集群使用逗号分割，例如--bootstrap-server 192.168.1.100:9092,192.168.1.101:9092
# --create              参数创建topic
# --delete              参数删除topic
# --list                参数查看topic
# --describe            参数查看topic详情
# --alter               参数修改topic配置
# --topic               参数指定topic名称，在创建、删除、修改、查看topic时会用到
# --partitions          参数设置主题的分区数（提高消费者并发），创建后可增加，但不能减少。3个分区表示可被3个消费者组同时消费，生产环境一般设置 3到12 个
#                       分区是什么？
#                       主题的物理分片，消息被分散存储在不同分区，分区越多，并发消费能力越强，吞吐量越高
# --replication-factor  参数设置每个分区的副本数（提高可用性），规则：副本数 不能超过 Broker 数量，单机 Kafka 只能设为 1，集群建议设为3表示每个分区有3个副本（1主2从）  
#                       副本是什么？
#                       分区的备份，用于保证高可用，一个 Leader + 多个 Follower
# --config retention.ms=86400000        # 设置消息保留时间为 1 天
# --config max.message.bytes=10485760   # 设置消息最大是 10MB
# --config min.insync.replicas=2        # 设置副本同步等待数               

# 查询主题
kafka-topics.bat --bootstrap-server localhost:9092 --list

# ​​启动生产者（Producer）​在 ​​生产者窗口​​ 输入消息，​​消费者窗口​​ 应能实时接收
bin\windows\kafka-console-producer.bat --bootstrap-server localhost:9092 --topic test 


# ​​启动消费者（Consumer）​
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
# 注意消息不是一旦消费就从Kafka中删除了，而是依旧存储Kafka按照顺序排好队，这样下一个消费者依旧可以找到历史消息进行消费
# --partitions 参数指定从哪个分区消费消息
# --from-beginning 参数表示从日志中最早的消息开始消费，而不是最新的消息。这样可以找到堆积的消息进行消费，而不是实时的消息消费
# --offset 参数指定从第几条消息开始消费（设置为0表示是从第1条消息开始消费，默认是latest表示忽略历史消息而监听实时消息进行消费）
# --group 参数设置消费者组（可以是数字或者其他的名称），一个消费者组中的多个消费者消费同一个topic时，则一条消息只会被消费者中的一个消费者消费，消费者组中的其他消费者不会重复消费这条消息。原因是一条消息在topic中只会被分配到一个partition中，而partition内部由offset指针划分前后消息是否消费，每个消费者组在partition内部都有一个专属于该消费者组的offset指针区分该消费者组已经消费的消息和未消费的消息
# --describe 参数可以查看消费者组的详细信息
~~~

### 依赖Zookeeper集群的Kafka3.8集群搭建
1. 搭建Zookeeper集群，参考Zookeeper集群搭建笔记
2. 到 https://kafka.apache.org/community/downloads/ 下载Kafka3.8解压安装包：将 Kafka 安装包解压至盘符根目录下并且复制3份，为每个节点重命名目录（如 D:\kafka-3.8-node1、D:\kafka-3.8-node2、D:\kafka-3.8-node3）
3. 修改各节点 server.properties：以 kafka-3.8-node1 为例，进入 D:\kafka-3.8-node1\config，编辑 server.properties：
~~~
# 节点broker ID编号，直接按照数字排列1、2、3
broker.id=1 

# 服务启动监听的节点ip地址和端口，如果只有一台服务器就需要更换3个端口9092、9093、9094
# PLAINTEXT 为明文协议，需指定节点 IP，避免使用 localhost和127.0.0.1，建议设置0.0.0.0监听所有网卡，包括回环和物理网卡，如果设置127.0.0.1连接只能走本地回环，但 Windows 对回环连接的清理机制很慢，大量连接关闭后停留在 CLOSE_WAIT 状态，无法释放。然后 Kafka 内部通信线程被阻塞Controller 不断重连，但连接一直处于半关闭状态，无法正常通信Windows 网络栈处理更稳定
listeners=PLAINTEXT://0.0.0.0:9092 

# 广告地址（客户端实际连接的地址，默认与 listeners 配置一致在linux无需设置，但是windows需要明确告诉其他节点：“请通过 127.0.0.1 连接我”，避免地址解析问题。）
advertised.listeners=PLAINTEXT://127.0.0.1:9092 

# 每个节点下的日志目录（也是消息存储目录），需要提前创建好
log.dirs=D:\kafka-3.8-node1\logs 

# zookeeper集群的连接地址，默认是注册到zookeeper的 / 根节点下，以下是设置注册到 /kafka 节点下
zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183/kafka 

# 每个 Topic 的默认分区数
num.partitions=3 

# 每个 Partition 的默认副本数（需小于等于 Broker 数量）
default.replication.factor=2 

# 数据保留时间（默认 7 天，可根据需求调整）
log.retention.hours=168

# 单个日志文件大小限制
log.segment.bytes=1073741824
~~~


4. 启动 Kafka 节点：分别打开 3 个 cmd 窗口，切换至各节点 bin\windows 目录，执行启动命令
~~~shell
# 若窗口显示 “Started KafkaServer” 则启动成功，注意启动过程中不要关闭 cmd 窗口（后台启动需通过脚本实现，测试环境可直接保持窗口打开）。
kafka-server-start.bat ..\..\config\server.properties
~~~

5. 集群功能验证
~~~shell
# 打开新 cmd 窗口，切换至任意 Kafka 节点的 bin\windows 目录，用 ZK 命令查看当前 Controller
# 需要检查所有节点的Controller
bin\windows\zookeeper-shell.bat localhost:2181 get /controller
# get /controller 是查看注册的controller节点，注意server.properties 中设置连接时有没有设置节点路径，上述配置是设置注册到 /kafka 节点下所以是 get /kafka/controller 
# 输出 {"version":1,"brokerid":1,"timestamp":"xxxx"} 可以看到brokerid

# 查看所有 Broker 是否都在集群中
zookeeper-shell.bat localhost:2181 ls /kafka/brokers/ids
# ls /brokers/ids 注意server.properties 中设置连接时有没有设置节点路径，上述配置是设置注册到 /kafka 节点下所以是 ls /kafka/brokers/ids
# 输出 [1, 2, 3] 可以看到 3 个节点都注册到集群

# 创建 Topic：打开新 cmd 窗口，切换至任意 Kafka 节点的 bin\windows 目录，执行：
kafka-topics.bat --create --topic disTopic --bootstrap-server 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --partitions 4 --replication-factor 2

# 查看 Topic 列表
kafka-topics.bat --bootstrap-server 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --list

# 查看 Topic 信息：
kafka-topics.bat --describe --topic disTopic --bootstrap-server 127.0.0.1:9092
# 主要输出如下
# Topic: disTopic TopicId: l6zebw9iS-2bP8DVMB6iPw PartitionCount: 4       ReplicationFactor: 2    Configs:
#         Topic: disTopic Partition: 0    Leader: 1       Replicas: 1,2   Isr: 1,2        Elr: N/A        LastKnownElr: N/A
#         Topic: disTopic Partition: 1    Leader: 2       Replicas: 2,3   Isr: 2,3        Elr: N/A        LastKnownElr: N/A
#         Topic: disTopic Partition: 2    Leader: 3       Replicas: 3,1   Isr: 3,1        Elr: N/A        LastKnownElr: N/A
#         Topic: disTopic Partition: 3    Leader: 1       Replicas: 1,3   Isr: 1,3        Elr: N/A        LastKnownElr: N/A
# 可以看到disTopic这个Topic有4个Partition，然后每个都有3个副本并且主副本和复制副本所在的节点


# 生产者（新 cmd 窗口）：输入一些消息
kafka-console-producer.bat --topic disTopic --bootstrap-server 127.0.0.1:9093

# 消费者（新 cmd 窗口）：启动后可以接收到消息
kafka-console-consumer.bat --topic disTopic --bootstrap-server 127.0.0.1:9094 --from-beginning
~~~

### Kafka可视化工具安装
- 安装offset Explorer可视化工具（原Kafka Tool），官网下载Offset Explorer，下载地址：https://www.kafkatool.com/download.html







