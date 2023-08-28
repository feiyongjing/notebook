### 参考官网： https://github.com/apache/dubbo 

### 开发文档：https://dubbo.apache.org/zh/docs/advanced/ 

### 设置负载均衡
参考： https://dubbo.apache.org/zh/docs/advanced/loadbalance/ 

在服务消费端设置@DubboReference的​loadbalance​的属性，默认是random​随机，消费端设置例子：
~~~java
@DubboReference(version = "${stock.server.version}",loadbalance = "roundrobin")
StockService stockService;
~~~
 

### 服务启动检查
参考：https://dubbo.apache.org/zh/docs/advanced/preflight-check/ 

服务启动时检查依赖的其他服务是否启动完成，若所依赖的其他服务未启动则启动失败，所以服务的启动是按照依赖的顺序来的，但是如果出现在两个或多个服务相互依赖的情况下启动顺序解决不了，这时候需要关闭启动启动检查，关闭方式如下：

关闭依赖的单个服务启动时检查
~~~java
@DubboReference(version = "${stock.server.version}",check = "false")
StockService stockService;
~~~
 

关闭所有依赖的服务启动时检查

~~~
dubbo.consumer.check=false
~~~