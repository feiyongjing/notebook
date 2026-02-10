### RabbitMQ简介
2007 年发布，是一个在AMQP(高级消息队列协议)基础上完成的，可复用的企业消息系统，是当前最主流的消息中间件之一。

优点：由于 erlang 语言的高并发特性，性能较好；吞吐量到万级，MQ 功能比较完备、健壮、稳定、易用、跨平台、支持多种语言如Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持 AJAX 文档齐全；开源提供的管理界面非常棒，用起来很好用，社区活跃度高；更新频率相当高。
缺点：商业版需要收费，学习成本较高。
选用场景：结合 erlang 语言本身的并发优势，性能好时效性微秒级，社区活跃度也比较高，管理界面用起来十分方便，如果你的数据量没有那么大，中小型公司优先选择功能比较完备的 RabbitMQ

# docker 安装RabbitMq
~~~shell
# 创建数据存储目录
mkdir -p /usr/local/docker/rabbitmq

# 持久启动，启动完成访问 http://127.0.0.1:15672 进行登录
docker run -id --name=rabbitmq -v /usr/local/docker/rabbitmq:/var/lib/rabbitmq -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:4-management
# -p 5672:5672：映射消息队列通信端口
# -p 15672:15672：映射管理界面端口
# -e RABBITMQ_DEFAULT_USER 和 RABBITMQ_DEFAULT_PASS：设置默认用户名和密码
# -v rabbitmq-data:/var/lib/rabbitmq：挂载数据卷以实现数据持久化

# 临时启动
docker run -id --name=rabbitmq -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:4-management

# 查看启动日志
docker logs -f rabbitmq
~~~


# RabbitMO的整体架构及核心概念
消息传输过程：一个RabbitMO中有多个虚拟主机，消息是先发送到其中一个虚拟主机中的交换机，交换机负责路由消息交给队列（可以同时路由给多个队列，如果交换机一个队列都没有绑定，那么发送到该交换机的消息默认会丢失，除非是做一些额外的配置），消费者从队列消费消息
- virtual-host：虚拟主机，起到数据隔离的作用
- exchange：交换机，负责路由消息
- queue：队列，存储消息
- publisher：消息发送者
- consumer：消息的消费者


## 交换机的类型有以下三种
- Fanout：广播类型的交换机，消息发送到广播类型的交换机，会传递给每个绑定该交换机的队列，也就是同一条消息传递到多个队列
- Direct：定向类型的交换机会将接收到的消息根据规则路由到指定的队列，因此称为定向路由。
    - 每一个队列都与定向类型的交换机绑定设置一个BindingKey
    - 发布者发送消息时，指定消息的RoutingKey
    - 定向类型的交换机将消息路由到BindingKey与消息RoutingKey一致的队列
- Topic：Topic交换机与Direct交换机类似，区别在于RoutingKey可以是多个单词的列表，并且以 . 分割，队列与交换机指定BindingKey时可以使用通配符:
    - #：代指0个或多个单词
    - *：代指一个单词





