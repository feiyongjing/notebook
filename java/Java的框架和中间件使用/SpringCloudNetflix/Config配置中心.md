### Spring Cloud Config为分布式系统中的外部化配置提供服务器和客户端支持。
Spring Cloud Config 是一种用来动态获取Git、SVN、本地的配置文件的一种工具。可以在所有环境中管理应用程序的外部属性。

可以与任何语言运行的任何应用程序一起使用。当应用程序通过部署从开发到测试并进入生产时，可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容。

服务器存储后端的默认实现使用git，可以轻松支持配置环境的标签版本，以及可用于管理内容的各种工具。添加替代实现并使用Spring配置插入很容易。

### Spring Cloud Config Server功能：
1.用于外部配置的HTTP，基于资源的API（名称 - 值对或等效的YAML内容）
2.加密和解密属性值（对称或非对称）
3.使用可轻松嵌入Spring Boot应用程序 @EnableConfigServer

### Spring Cloud Config Client功能（适用于Spring应用程序）：
1.绑定到Config Server并Environment使用远程属性源初始化Spring
2.加密和解密属性值（对称或非对称）

# 使用Spring Cloud Config

### 总共有三大步骤，第二步和第三步在下面详细说明

1. 创建版本控制仓库存放配置文件，是专门创建一个仓库，不要放其他的东西否则服务端会将仓库的其他东西也一并拉取到本地
2. 配置Spring Cloud Config服务端
3. 配置Spring Cloud Config客户端
---

## 配置Spring Cloud Config服务端详情

### 1. 创建一个新的Springboot项目，用于启动配置中心服务

### 2. 添加Spring Cloud Config依赖和eureka的依赖
~~~xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-config-server</artifactId>
     <version>2.2.8.RELEASE</version>
</dependency>
~~~

### 3. 添加yml配置
~~~yml
server:
  port: 8888 # 服务端口

spring:
  application:
    name: springcloud-config-server # 当前服务的名字
  cloud:
    config:
      server:
        git:
          uri: https://github.com/feiyongjing/spring-cloud-project-template.git # config配置所在的git仓库地址
          search-paths: springcloud-config # 远程仓库下config配置文件所在的的目录
#          force-pull: true       # 强行pull拉取
#          username: feiyongjing  # 填写登录用户名，公开的库不用写
#          password: "填写密码"    # 填写登录密码，公开的库不用写
~~~

### 4. 启动类添加@EnableConfigServer

### 5. 启动Spring Cloud Config服务端项目，在本地会有一份远程仓库的复制，通过多种接口可以获取配置文件

接口一：/{application}/{profile}[/{label}]         例子：http://localhost:8888/application/dev/main

接口二：/{label}/{application}-{profile}.properties      例子：http://localhost:8888/main/application-dev.properties

 解释：application是配置文件的第一个单词，一般情况下就是application本身

            profile是环境，有dev、test、online及默认

            label表示配置文件仓库的分支，例如master

            接口二的文件后缀名是可以使用yml的
---

## 配置Spring Cloud Config客户端详情

### 1. 给需要获取配置的客户端项目添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>2.2.8.RELEASE</version>
</dependency>
~~~

### 2. 对客户端项目下的resources资源文件下创建一个bootstrap.properties文件或者是yml文件，文件内容如下
~~~yml
spring:
  application:
    name: application  # 对应Config服务端接口路径中的application
  cloud:
    config:
      uri: http://localhost:8888/  # 对应Config服务端接口的域名和端口
      profile: dev                 # 对应Config服务端接口路径中的profile，即环境
      label: main                  # 对应Config服务端接口路径中的label，即远程仓库的标签，一般是分支
~~~

### 3. 将原来的配置文件剪切上传到配置文件远程仓库中

### 4. 提前启动启动Spring Cloud Config服务端项目，然后启动Spring Cloud Config客户端项目

 

# 远程仓库配置文件敏感信息加密

### 1. 在orcale下载JCE，解压缩后复制压缩文件中的全部  .jre 文件粘贴到 D:\software\Java\Jdk-8u291\jre\lib\security 目录下

### 2. 在Config服务端配置密钥，即在Spring Cloud Config服务端项目下的resources资源文件下创建一个bootstrap.properties文件或者是yml文件，文件内容如下
~~~yml
encrypt:
  key: ha9ph3@$%&*%@%$(@#%E!  # 随机编写的对称密钥
~~~

### 3. 加密接口Post类型：http://localhost:8888/encrypt，通过加密接口发送原始数据获取原始数据加密后的数据
解密接口Post类型：http://localhost:8888/decrypt，通过解密接口发送加密的数据获取原始的数据
通过Postman或者是IDEA请求脚本获取加密后的数据，一次只能对一条数据加密，多个数据需要多次请求获取数据
Postman发送请求时将原始的数据放入请求体，使用raw类型以text方式（加解密都一样）
若使用IDEA脚本则声明Content-Type: application/text​，原始数据直接放入原JSON的位置即可（加解密都一样）

客户端项目的配置文件使用加密后的数据来替换原来的数据，例如原来的数据库连接配置如下
~~~yml
spring:
  datasource:
    name: root
    password: 123456
~~~

替换后的配置如下，注意{cipher}是告诉spring后面的数据是加密后的密文
~~~yml
spring:
  datasource:
    name: '{cipher}8b13705c5cd815f2c366d1b747e2338a48979dc0815d43e591285de0a9e058b4'          # 数据库用户名加密后的密文
    password: '{cipher}53f3e4940671b09444dc4d2f9bd8bce920f172d3999ed5b05c5537ac0d50b865'      # 数据库用户密码加密后的密文
~~~
 

# 配置信息局部刷新

### 1. 给需要获取配置的客户端项目添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

### 2. 在客户端项目读取配置的类上添加@RefreshScope，即含有被@Value读取的属性类或者是其他读取方式的类，添加了该注解的类会在配置更新是得到特殊的处理

### 3. 客户端项目添加yml配置，打开web访问端点
~~~yml
management:
  endpoints:
    web:
      exposure:
        include: '*'  # 如果直接给*，就会打开所有的Web访问端点
~~~

### 4. 重启客户端，访问客户端的接口（post请求）：http://localhost:8080/actuator/refresh  进行手动刷新配置，并且只能一个一个的刷新微服务配置，如果微服务多就会显得很蠢

 

#  配置信息全局刷新

### 1. 给Spring Cloud Config服务端项目添加bus依赖和actuator依赖
~~~xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.2.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

### 2. Spring Cloud Config服务端项目配置中添加RabbitMQ的配置，以及打开Web访问端点
~~~yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin

management:
  endpoints:
    web:
      exposure:
        include: '*'  # 如果直接给*，就会打开所有的Web访问端点
~~~

### 3. Spring Cloud Config客户端项目配置中添加RabbitMQ的配置，注意是远程仓库中的配置文件修改
~~~yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin
~~~

### 4. 启动RabbitMQ服务 

### 5. 重启客户端和服务端，修改配置后上传到远程仓库，然后访问服务端的接口（post请求）：http://localhost:8888/actuator/bus-refresh 来刷新配置
请求返回1代表请求刷新成功，RabbitMQ会收到消息，所有的客户端都会接收消息然后刷新配置

 

# Spring Cloud Config 高可用
 
## 方式一：保证Spring Cloud Config 高可用只需要将Spring Cloud Config 服务端集群注册到注册中心就行了

### 1. Spring Cloud Config 服务端添加eureka client依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>2.2.10.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
        </exclusion>
    </exclusions>
</dependency> 
~~~

### 2. Spring Cloud Config 服务端配置文件添加eureka的配置
~~~yml
eureka:
  instance:
    lease-renewal-interval-in-seconds: 2 
    lease-expiration-duration-in-seconds: 10 
    prefer-ip-address: true 
    instance-id: springcloud-config-server-8888 
  client:
    service-url:
      defaultZone: http://eureka8762:8762/eureka,http://eureka8763:8763/eureka,http://eureka8764:8764/eureka 
    eureka-server-connect-timeout-seconds: 30
    eureka-server-read-timeout-seconds: 60
~~~

### 3. Spring Cloud Config 客户端启动的 bootstrap.yml 配置文件添加配置
~~~yml
spring:
  cloud:
    config:
      discovery:
        enabled: true # 让配置中心的url可以被注册中心发现
        service-id: springcloud-config-server # 配置中心在注册中心注册的服务名称

eureka:
  client:
    service-url:
      defaultZone: http://eureka8762:8762/eureka,http://eureka8763:8763/eureka,http://eureka8764:8764/eureka  # 注册中心集群地址
~~~
---

## 方式二：使用nginx做负载均衡转发请求

### 1. 启动nginx服务

### 2. 修改Spring Cloud Config 客户端启动的 bootstrap.yml 配置文件属性
~~~yml
 spring:
  cloud:
    config:
      uri: http://localhost:8888/   # 这里修改使用nginx来进行转发到达高可用
~~~

# Spring Cloud Config 安全认证
给Spring Cloud Config 服务端的接口添加安全认证，不让其他人随便的就获取到配置文件

### 1. Spring Cloud Config 服务端项目添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.3.12.RELEASE</version>
</dependency>
~~~

### 2. Spring Cloud Config 服务端项目添加配置属性
~~~yml
spring:
  security:
    user:
      name: admin             # 服务端设置用户名
      password: 123456        # 服务端设置密码
~~~

### 3. Spring Cloud Config 服务端项目 启动配置boodstrap.yml 添加配置属性
~~~yml
spring:
  cloud:
    config:
      username: admin         # 客户端访问服务端的用户名
      password: 123456        # 客户端访问服务端的密码
~~~

 