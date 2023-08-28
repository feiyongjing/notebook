### CAP定理
在一个分布式系统中，以下三点特性无法同时满足，「鱼与熊掌不可兼得」
一致性（C）：
在分布式系统中的所有数据备份，「在同一时刻是否拥有同样的值」。（等同于所有节点访问同一份最新的数据副本）

可用性（A）：
在集群中一部分节点「故障」后，集群整体「是否还能响应」客户端的读写请求。（对数据更新具备高可用性）

分区容错性（P）：
即使出现「单个组件无法可用,操作依然可以完成」  

### Nacos可以在cp和ap之间切换，默认是ap

Nacos 客户端使用心跳上报方式维持活性，发送心跳的周期默认是 5 秒，Nacos 服务端会在 15 秒没收到心跳后将实例设置为不健康，在 30 秒没收到心跳时将这个服务实例摘除。

对于服务的保护阈值设置在0到1之间，一个服务的不健康实例除以这个服务的总实例数如果小于该服务的阈值则会将不健康的实例也拿去使用抗压，但是不健康的实例不稳定会使访问超时进而导致服务雪崩，所以一般服务的保护阈值不设置默认是0

服务的权重是用于负载均衡的，即权重越大则被访问的概率越大

 

# 启动单机版Nacos注册中心
1. Nacos 依赖 Java 环境来运行，可以从 最新稳定版本 选择下载需要的版本 nacos-server-$version.zip 包，解压缩后进入 nacos/bin 目录下

2. 根据系统环境来启动Nacos，默认是集群启动，开发使用单机启动，注意一定要按照系统环境来启动，不然启动之后无法登录控制台
~~~shell
# Linux/Unix/Mac
# 启动命令(standalone代表着单机模式运行，非集群模式):
sh startup.sh -m standalone
# 如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：
bash startup.sh -m standalone

# Windows
# 启动命令(standalone代表着单机模式运行，非集群模式):
startup.cmd -m standalone﻿​
~~~
3. 访问控制台 http://192.168.0.100:8848/nacos/index.html 进行登录，默认用户名和密码都是nacos

# 启动Nacos注册中心集群
1. Nacos 依赖 Java 环境来运行，可以从 最新稳定版本 选择下载需要的版本 nacos-server-$version.zip 包，解压缩后的文件复制多份

2. 在每个解压缩文件中 nacos/conf目录下对application.properties.example 文件复制粘贴到原目录，重命名为application.properties，并修改内容，如下
~~~
### Default web server port:
server.port=8849  # 修改注册中心启动的端口
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql # 使用什么数据源

### Count of DB:
db.num=1 # 数据源的数量

### Connect URL of DB 连接数据源的详细信息
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=123456
~~~
3. 在每个解压缩文件中 nacos/conf目录下对cluster.conf.example 文件复制粘贴到原目录，重命名为cluster.conf，并修改内容，如下 
~~~
192.168.0.100:8849 #添加整个注册中心集群的Ip和端口
192.168.0.100:8850
192.168.0.100:8851
~~~
4. 在每个解压缩文件中 nacos/bin目录下使用命令启动注册中心，不同的环境启动命令不同，如下
~~~shell
# Linux/Unix/Mac
sh startup.sh 
# 如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：
bash startup.sh

# Windows
startup.cmd 
~~~
5. 启动后访问控制台：http://192.168.0.101:8849/nacos/index.html  默认的用户名和密码都是nacos

### 命名空间与组名的介绍： https://blog.csdn.net/leilei1366615/article/details/111405644 

# 微服务客户端连接Nacos注册中心、注册和发现服务
1. 客户端项目添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
~~~
2. 客户端项目添加配置
~~~yml
spring:
  application:
    name: order-server # 服务的名字
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # 连接的注册中心地址和端口，如果是集群使用逗号分隔
        username: nacos # 连接注册中心地址登录用户名
        password: nacos # 连接注册中心地址登录密码
        namespace: public  # public是默认的命名空间，这里一般是命名空间的ID，注意服务的提供者和调用者的命名空间保持一致
        phemeral: false # 默认是true临时实例，而false是永久实例，即服务挂掉注册中心也不会清除服务实例
        # group: DEFAULT_GROUP # 对服务进行的分组，现在是默认的分组，注意服务的提供者和调用者的组名保持一致
        # weight: 0 # 服务权重设置
~~~
3. 在启动类上添加@EnableDiscoveryClient​注解，注意在高版本的SpringBoot中不需要添加注解

# 注意服务的提供者和调用者的命名空间与分组都要保证一致，不然就无法发现服务进行调用
 
