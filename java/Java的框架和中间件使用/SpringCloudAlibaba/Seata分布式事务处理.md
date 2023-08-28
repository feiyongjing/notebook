### 2PC相关请查看分布式事务介绍相关笔记
官方文档：https://seata.io/zh-cn/docs/overview/what-is-seata.html

# Seata的模式有AT模式、TCC模式等

### AT模式（Auth transaction）是对2PC的实现解决，分为两个阶段
- 在第一个阶段中会拦截所有的业务SQL，解析SQL语义，找到业务SQL需要更新的原数据保存到“before image”，然后执行义务SQL更新业务数据，更新后的业务数据保存到“after image”，最后对更新的数据生成行锁使其他的SQL无法读写这些数据，并提交事务。以上操作是在一个数据库事务中完成，保证了一阶段操作的原子性

- 第二个阶段如果是提交事务，则删除“before image”、“agter image”，并解开一阶段生成的行锁，因为一阶段已经提交事务了所有无需再进行提交事务

- 第二个阶段如果是回滚事务，首先进行数据校验，即现在从数据库查找到数据与“after image”的数据对比，但是数据上了行锁，所有对比的结果一定是一致的。

- 然后根据“before image”的原数据与第一阶段执行的业务SQL拼装出逆向SQL执行，达到进行数据还原。

- 最后删除“before image”、“agter image”，并解开一阶段生成的行锁，因为一阶段已经提交事务了所有无需再进行提交事务
---
 
# Seata角色
### Seata分TC、TM和RM三个角色，TC（Server端）为单独服务端部署，TM和RM（Client端）由业务系统集成。
- TC (Transaction Coordinator) - 事务协调者，维护全局和分支事务的状态，驱动全局事务提交或回滚  
- TM (Transaction Manager) - 事务管理器，定义全局事务的范围：开始全局事务、提交或回滚全局事务  
- RM (Resource Manager) - 资源管理器，管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚  

 

# 使用Seata
### 使用Seata需要搭建Seata TC(Server端)，TM和RM（Client端）即业务系统的配置集成。

# 搭建Seata TC(Server端)
### 1.下载Seata Server对应版本启动包：https://github.com/seata/seata/releases 并解压缩
### 2. 启动Seata Server 集群
由于Seata Server端存储模式（store.mode）现有file、db、redis三种（后续将引入raft,mongodb）
- file模式无需改动，直接启动即可，注： 全局事务会话信息内存中读写并持久化本地文件root.data，性能较高，但是由于使用内存和数据本地本地持久化，所以只能单机启动而无法集群启动
- db模式为高可用模式，全局事务会话信息通过db共享，相应性能差些;
- redis模式Seata-Server 1.3及以上版本支持,性能较高,存在事务信息丢失风险,请提前配置合适当前场景的redis持久化配置.  
---
现在一般使用的是db数据库模式，首先要启动数据库服务，然后使用数据库连接工具创建一个数据库，例如例子中数据库的名字就叫seata
在 https://github.com/seata/seata 选择与第一步Seata相同的版本下载zip包，解压缩后在/script/server/db目录下有不同的数据库SQL文件，选择对应的数据库SQL文件
在新建的数据库下执行SQL文件内容，执行完后数据库中有lock_table、branch_table 与 global_table三张表
- global_table存储事务的全局事务信息
- branch_table存储事务的分支信息
- lock_table存储事务的锁住的信息
---
打开第一步解压缩后的 /conf目录下的file.conf 文件修改内容，mode = "db"，还有连接数据库的URL和用户名和密码
由于防止Seata Server单点故障而使用集群部署Seata Server，所以Seata Server需要注册到注册中心进行管理，这里使用注册中心是Nacos，同时也对Seata Server的配置文件进行了管理，配置方式如下
打开第一步解压缩后的 /conf目录下的registry.conf 文件修改内容，需要修改的属性介绍如下
~~~
registry {         # 这是连接注册中心的配置
  type = "nacos"   # 选择注册中心的类型

  nacos {
    application = "seata-server"  # 启动seata集群的名字
    serverAddr = "localhost:8849,localhost:8850,localhost:8851"  # 连接的注册中心地址和端口
    group = "seata-group"  # 对seata集群的分组
    namespace = "774b7a5e-df22-4173-9c39-45fd0f9b7adc" # 命名空间的ID
    cluster = "default" # Nacos集群的名字
    username = "nacos" # 连接注册中心地址登录用户名
    password = "nacos" # 连接注册中心地址登录密码
  }
}

config {         # 这是连接配置中心的配置
  type = "nacos" # 选择配置中心的类型

  nacos {
    serverAddr = "localhost:8849,localhost:8850,localhost:8851" # 连接的注册中心地址和端口
    namespace = "774b7a5e-df22-4173-9c39-45fd0f9b7adc"  # 命名空间的ID
    group = "seate" # 配置文件的分组名
    username = "nacos" # 连接注册中心地址登录用户名
    password = "nacos" # 连接注册中心地址登录密码
  }
}
~~~
---

在 https://github.com/seata/seata 选择与第一步Seata相同的版本下载zip包，解压缩后在/script/config-center目录下打开config.txt文件修改指定的属性
~~~
# my_test_tx_group是自定义的组名一般是地名，目的是对Seata Server集群分组在不同的异地部署达到容灾的效果
service.vgroupMapping.my_test_tx_group=default # default是指Nacos集群的名字

store.mode=db
store.db.driverClassName=com.mysql.cj.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3307/seata?useUnicode=true
store.db.user=root
store.db.password=123456

# 下面是例子
service.vgroupMapping.beijing=default
service.vgroupMapping.shanghai=default
~~~
---


打开在 https://github.com/seata/seata 选择与第一步Seata相同的版本下载zip包，解压缩后/script/config-center/nacos目录运行nacos-config.py Python脚本或者是nacos-config.sh脚本将上述/script/config-center目录下的config.txt文件上传到Nacos配置中心，启动命令参数详解如下
~~~shell
bash nacos-config.sh -h 127.0.0.1 -p 8848 -g seata -t 774b7a5e-df22-4173-9c39-45fd0f9b7adc -u nacos -w nacos
# -h 指定Nacos服务启动主机IP地址，默认是localhost
# -p 指定Nacos服务启动的端口号，默认是8848
# -g 配置分组组名，默认是SEATA-GROUP
# -t Nacos中不同命名空间ID，默认值是空
# -u 连接Nacos配置中心地址登录用户名
# -w 连接Nacos配置中心地址登录密码
~~~
---

打开第一步解压缩后的 /bin目录执行命令启动Seatea Server，Linux环境启动seata-server.sh，windows环境启动seata-server.bat，启动参数如下
~~~shell
seata-server.bat -p 8092 -h 192.168.0.101 
# -h 指定在注册中心注册的 IP，不指定时获取当前的 IP可能不是IP V4的地址，外部访问部署在云环境和容器中的 server 建议指定
# -p 指定Seata server启动的端口号，默认是8091
# -m 事务日志存储方式，支持file,db,redis，默认为 file 注:redis需seata-server 1.3版本及以上
# -n 用于指定seata-server节点ID，如 1,2,3..., 默认为 1
# -e 指定 seata-server 运行环境，如 dev, test 等, 服务启动时会使用 registry-dev.conf 这样的配置
~~~
---
 

# 搭建Seata Client端
### 1. 在涉及到分布式事务相关所有的微服务项目添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
~~~
---

### 2. 在所有涉及到分布式事务相关微服务项目连接的数据库中创建如下数据表，数据表的来源是：
打开在 https://github.com/seata/seata 选择与第一步Seata相同的版本下载zip包，解压缩后/scripe/client/at/db目录下选择对应的数据库文件执行SQL

目前是1.3.0版本的数据表

~~~sql
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `branch_id`     BIGINT(20)   NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(100) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME(6)  NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME(6)  NOT NULL COMMENT 'modify datetime',
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';

~~~
---

### 3. 在所有涉及到分布式事务相关微服务项目添加配置属性
~~~yml
spring:
  cloud:
    alibaba:
      seata:
        tx-service-group: my_test_tx_group  # 是上述Seata TC搭建步骤中上传到Nacos配置文件内容，值自定义的组名一般是地名，目的是对Seata Server集群分组在不同的异地部署达到容灾的效果
        
seata:
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8849
      username: nacos
      password: nacos
      application: seata-server
      group: DEFAULT_GROUP
      namespace: 774b7a5e-df22-4173-9c39-45fd0f9b7adc
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8849
      username: nacos
      password: nacos
      group: seata
      namespace: 00811662-3dea-4999-a45b-b87dfdd78118
~~~
---


### 4. 在项目中使用@GlobalTransactional﻿​替换原有的@Transactional﻿​
