Sleuth是一款分布式的链路跟踪系统，跟踪生成一些数据，这些数据不便于人类阅读，所以会将这些数据上传到Zinkin Server
Zipkin Server 通过页面统一进行数据的展示

## 搭建Zipkin Server
### 1. 创建一个Spring Boot 项目

### 2. 添加依赖
~~~xml
<dependency> 
    <groupId>io.zipkin.java</groupId> 
    <artifactId>zipkin-autoconfigure-ui</artifactId> 
    <version>2.12.9</version> 
</dependency> 
<dependency> 
    <groupId>io.zipkin.java</groupId> 
    <artifactId>zipkin-server</artifactId> 
    <version>2.12.9</version> 
</dependency>
~~~
### 3. 添加配置
~~~yml
server:
  port: 9411 # 服务端口

spring:
  application:
    name: springcloud-zipkin # 当前服务的名字

management:
  metrics:
    web:
      server:
        request:
          autotime:
            enabled: false # 关闭自动配置启用所有请求的检测
~~~

### 4. 启动类上添加@EnableZipkinServer注解

### 5. 启动项目访问http://localhost:9411

## 注意在Spring Boot 的版本大于2就会出现版本问题需要下载jar包或者是Docker方式启动

### docker方式启动Zipkin
~~~shell
docker run -d --name zipkin-server -p 9411:9411 openzipkin/zipkin
~~~

如果需要整合elasticsearch，则需要先启动elasticsearch服务
~~~shell
docker run -d --name elasticsearch-server -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.16.1
~~~
并且zipkin服务启动添加-e STORAGE_TYPE=elasticsearch -e ES_HOSTS=192.168.0.101:9200 ，注意-e环境其中ES服务的ip不能使用localhost
~~~shell
docker run -d --name zipkin-server -e STORAGE_TYPE=elasticsearch -e ES_HOSTS=192.168.0.101:9200 --restart always -p 9411:9411 openzipkin/zipkin
~~~


# 微服务整合Sleuth

### 对需要监控的微服务都进行如下操作

### 1. 添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
    <version>2.2.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
    <version>2.2.8.RELEASE</version>
</dependency>
~~~

### 2. 对需要监控的项目添加配置
~~~yml
spring:
  zipkin:
    base-url: http://localhost:9411 # 指定Zipkin server 的地址
    sender:
      type: web  # 发送跟踪数据到Zipkin的类型web(http)
  sleuth:
    sampler:
      probability: 1.0 # 对于微服务接收的请求采样的数量，默认是0.1，也就是10%，因为分布式系统中采集庞大的数据量会影响性能，这里根据数据量来设置
~~~
