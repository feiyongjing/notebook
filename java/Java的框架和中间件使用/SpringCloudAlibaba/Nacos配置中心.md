### 与SpringCloud Config一样都是对微服务配置文件进行管理，但是相对于SpringCloud Config的好处是提供了可视化的界面，修改配置后不需要刷新Bus消息总线
 
# 使用Nacos配置中心

### 1. 在Nacos控制台的配置管理的配置列表中选择一个 nameSpace 新建配置，注意data ID一般是客户端的项目名字、group是对同一项目配置文件的分组，添加配置文件

~~~
data ID正常情况下的完整格式：${prefix}-${spring.profiles.active}.${file-extension}

${prefix}是项目的名字

${spring.profiles.active}是项目的环境，如果在第三步的bootstrap.yml配置中没有设置data ID就可以直接省略为项目的名字  

${file-extension}是配置中心存放的配置文件后缀，如果上述的${spring.profiles.active}设置了则一定要添加配置文件的后缀
~~~

注意下面客户端自定义读取不同data ID配置时可以完全不按照上述规则为data ID命名

官方参考：https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html  

图文详细参考：https://www.cnblogs.com/konglxblog/p/15201270.html

### 2. 客户端添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
~~~
### 3. 创建bootstrap.yml 配置文件，并添加配置
~~~yml
spring:
  application:
    name: order-server # 程序的名字必须指定，原因是与第一步的Data ID 有关联
#  profiles:
#    active: dev # 配置文件的环境声明，与第一步的Data ID 有关联
  cloud:
    nacos:
      config:
        server-addr: localhost:8849,localhost:8850,localhost:8851 # 连接的注册中心地址和端口
        username: nacos # 连接注册中心地址登录用户名
        password: nacos # 连接注册中心地址登录密码
        namespace: 774b7a5e-df22-4173-9c39-45fd0f9b7adc  # 除了默认的空间命名是public，之外的命名空间都是命名空间对应ID
        file-extension: yaml # 默认读取的是properties类型的配置文件，使用其他类型需要添加
        group: order # 项目分组的组名
#        refresh-enabled: false # 是否自动获取最新的配置，默认是true
~~~

### 4. 检查客户端项目是否是读取配置后正常启动，注意如果是通过@Value读取配置属性需要在对应的类上添加@RefreshScope，例子如下
~~~java
@RestController
@RefreshScope
public class OrderController {

    @Value("${abc}")
    String abc;
    
    @RequestMapping("/test")
    public String test(){
        return abc;
    }
~~~
 

# 客户端自定义读取不同data ID配置（可同时读取多个配置文件）
shared-configs和extension-configs都可以设置选择读取多个配置文件，在bootstrap.yml 配置文件中配置，如下

注意不同的配置方式让读取多个配置文件会让配置文件中相同的配置属性覆盖，覆盖规则是优先级高的覆盖优先级低的，如下

### 完整配置命名data ID的配置>默认与项目名一致的data ID配置>extension-configs设置的自定义data ID配置>shared-configs设置的自定义data ID配置
~~~yml
spring:
  application:
    name: order-server
  cloud:
    nacos:
      server-addr: localhost:8849,localhost:8850,localhost:8851 # 连接的注册中心地址和端口
      username: nacos # 连接注册中心地址登录用户名
      password: nacos # 连接注册中心地址登录密码
      config:
        namespace: 774b7a5e-df22-4173-9c39-45fd0f9b7adc  # 以命名来隔离不同的服务，用于不同的环境
        shared-configs:         # shared-configs可以读取多个自定义的data ID和group的配置文件并自动刷新配置 
          - data-id: xxx0
            refresh: true
            group: yyy0
          - data-id: xxx1
            refresh: true
            group: yyy1
        extension-configs:      # extension-configs可以读取多个自定义的data ID和group的配置文件并自动刷新配置
          - data-id: aaa0
            refresh: true
            group: bbb0
          - data-id: aaa1
            refresh: true
            group: bbb1
~~~

### 对配置文件进行权限控制
1. 在Nacos下载解压缩后的文件中 nacos/conf目录下对application.properties修改内容，如下
~~~
nacos.core.auth.enabled=true
~~~
2. 重启Nacos服务，访问控制台就可以使用权限控制功能了
