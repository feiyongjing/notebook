### Spring Cloud Stream 是一个构建消息驱动微服务的框架。

Spring Cloud Stream构建在SpringBoot之上，提供了Kafka，RabbitMQ等消息中间件的个性化配置，引入了发布订阅、消费组和分区的语义概念，有效的简化了上层研发人员对MQ使用的复杂度，让开发人员更多的精力投入到核心业务的处理。

在实际开发过程中，服务与服务之间通信经常会使用到消息中间件，而以往使用了哪个中间件比如RabbitMQ，那么该中间件和系统的耦合性就会非常高，如果我们要替换为Kafka那么变动会比较大，使用Spring Cloud Stream来整合我们的消息中间件，可以降低系统和中间件的耦合性。

### 应用模型
Spring Cloud Stream由一个中立的中间件内核组成。
Spring Cloud Stream会注入输入和输出的channels(信道)，应用程序通过这些channels与外界通信，而channels则是通过Binder与外部消息中间件连接。

# 项目中使用Stream
消息生产者和消息消费者项目都有修改

## 消息生产者项目使用

### 第一步：添加依赖
~~~xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>   <!-- 这是集成rabbitMQ的stream, 使用其他的消息中间件需要替换 -->
    <version>3.0.13.RELEASE</version>
</dependency>
~~~

### 第二步：修改yml配置
~~~yml
spring:
  cloud:
    stream:
      binders:
        rabbitmq:         # 这个rabbitmq是binders中String类型key,是自定义的，会在与bindings绑定时使用
          type: rabbit
          environment:
            spring:       # 这个spring是environment中String类型key,是自定义的
              rabbitmq:
                host: localhost
                prot: 5672
                username: admin
                password: admin
                virtual-host: / # 虚拟主机，在rabbitmq中不同的虚拟主机exchange、queue、message不能互通，相当于数据库的分库
      bindings:
        output:           # 这个output是管道名,对应在@EnableBinding({Source.class})中Source.class的OUTPUT属性，或者自己写的Source.class接口OUTPUT属性值
          destination: spring.cloud.stream.exchange # 定义交换机的名称
          binder: rabbitmq  # 这个rabbitmq是spring.cloud.stream.binders所自定义设置的key
~~~

### 第三步：通过默认的Source.class绑定和获取channels信道发送消息
~~~java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;

// 将channels(信道)和exchange(交换机)通过默认的Source.class绑定在一起
@EnableBinding({Source.class})
public class SenderMessage {
    @Autowired
    private MessageChannel output; // 消息发送的信道，jar包处理好了直接注入就行

    public void publish(String msg){
        boolean flag = output.send(MessageBuilder.withPayload(msg).build());
        System.out.println("发送消息："+msg+"是否成功："+flag);
    }
}
~~~

### 如果需要自定义Source.class使用，首先创建一个自定义的Source接口，例子如下
~~~java
import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

public interface MySource {

    String OUTPUT = "myoutput";

    @Output(MySource.OUTPUT)
    MessageChannel output();
}
~~~

### 然后修改第二步的配置
~~~
spring.cloud.stream.bindings.myoutput # 替换spring.cloud.stream.bindings.output 
~~~

### 再修改第三步的代码
~~~java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.messaging.support.MessageBuilder;

// 将channels(信道)和exchange(交换机)绑定在一起
@EnableBinding({MySource.class})
public class SenderMessage {
    @Autowired
    private MySource mySource;

    public void publish(String msg){
        boolean flag = mySource.output().send(MessageBuilder.withPayload(msg).build());
        System.out.println("发送消息："+msg+"是否成功："+flag);
    }
}
~~~
---

## 消息消费者项目使用

### 第一步：添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>   <!-- 这是集成rabbitMQ的stream, 使用其他的消息中间件需要替换 -->
    <version>3.0.13.RELEASE</version>
</dependency>
~~~

### 第二步：修改yml配置
~~~yml
spring:
  cloud:
    stream:
      binders:
        rabbitmq:         # 这个rabbitmq是binders中String类型key,是自定义的，会在与bindings绑定时使用
          type: rabbit
          environment:
            spring:       # 这个spring是environment中String类型key,是自定义的
              rabbitmq:
                host: localhost
                prot: 5672
                username: admin
                password: admin
                virtual-host: / # 虚拟主机，在rabbitmq中不同的虚拟主机exchange、queue、message不能互通，相当于数据库的分库
      bindings:
        input:           # 这个input是管道名,对应在@EnableBinding({Sink.class})中Sink.class的INPUT属性，或者自己写的Sink.class接口INPUT属性值
          destination: spring.cloud.stream.exchange # 定义交换机的名称
          binder: rabbitmq  # 这个rabbitmq是spring.cloud.stream.binders所自定义设置的key
~~~

### 第三步：接收消息处理
~~~java
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.Message;

@EnableBinding(Sink.class) // 将channels(信道)和exchange(交换机)通过默认的Sink.class绑定在一起
public class MessageReceiver {

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        System.out.println("接收消息处理："+message.getPayload());
    }
}
~~~

### 如果需要自定义Sink.class使用，首先创建一个自定义的Sink接口，例子如下
~~~java
import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

public interface MySink {
    String INPUT = "myinput";
    
    @Input(MySink.INPUT)
    SubscribableChannel input();
}
~~~

### 然后修改第二步的配置
~~~
spring.cloud.stream.bindings.myinput # 替换spring.cloud.stream.bindings.input 
~~~

### 再修改第三步的代码
~~~java
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.Message;

// 将channels(信道)和exchange(交换机)绑定在一起
@EnableBinding(MySink.class)
public class MessageReceiver {

    @StreamListener(MySink.INPUT)
    public void input(Message<String> message){
        System.out.println("接收消息处理："+message.getPayload());
    }
}
~~~

## 消息的分组与持久化

### 分组的作用

第一是实现消息的持久化，即消费者挂掉后重启依然可接收之前生产者生产的消息。

第二是实现一条消息只能被一个消费者接收消费，即应用于消费者集群部署时。

只需要在消费端项目添加配置就可以对消息进行分组，队列名称是交换机的名字加上destination﻿​.group
~~~
spring.cloud.stream.bindings.myinput.group=rabbitmq-group # 注意myinput是自定义的管道名 
~~~

设置消费者绑定的路由key
~~~
spring.cloud.stream.rabbit.bindings.myinput.comsumer.bindingRoutingKey='spring.cloud.stream.#' # 设置消费者绑定的路由key 
~~~
 