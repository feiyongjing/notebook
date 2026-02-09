## SpringBoot集成RabbitMq
1. 引入依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!-- 对象发送序列化 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
~~~

2. 配置文件添加配置
~~~yml
spring:
  rabbitmq:
    host: "localhost"
    port: 5672
    username: "admin"
    password: "admin"
    virtual-host: "/"
    listener:
      simple:
        # 每次只取1条消息消费，处理完成后会立刻取下一条消息消费
        # 默认消息是发送绑定在队列的每个消费者是轮询的，没有考虑消费者消费速度差异，消费速度快和慢的消费者消费的速率是一样的，容易产生消息堆积问题，通过设置这个可以让消费速度快的消费者提供消费速率
        prefetch: 1
        connection-timeout: 1s      # 连接超时时长
    template:
      # 当网络不稳定的时候，利用重试机制可以有效提高消息发送的成功率。
      # 不过SpringAMQP提供的重试机制是阻塞式的重试，也就是说多次重试等待的过程中，当前线程是被阻塞的，会影响业务性能。
      # 如果对于业务性能有要求，建议禁用重试机制。如果一定要使用，请合理配置等待时长和重试次数，当然也可以考虑使用异步线程来执行发送消息的代码。
      retry:
        enabled: true                   # 开启超时重试
        initial-interval: 200ms         # 失败后的等待时间段
        multiplier: 1                   # 失败后下次的等待时长倍数，下次等待时长 = initial-interval * multiplier
        max-attempts: 3                 # 最大重试次数
    publisher-confirm-type: correlated  # 默认none:关闭confirm机制，simple:同步阻塞等待MQ的返回结果，correlated:MQ异步回调方式返回结果，通过回执消息可以进行判断ack是否ok
    publisher-returns: true             # 默认false关闭消息返回机制，无法路由到队列的消息直接丢弃（不会返回给生产者，但是ack是返回ok的）
~~~

3. 添加序列化
~~~java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JacksonConfig {

    // 默认的JDK序列化缺点：包含元数据体积过大、有安全漏洞、二进制数据可读性差，需要使用其他的json序列化器
    @Bean
    public Jackson2JsonMessageConverter jsonMessageConverter() {
        // LaissezFaireSubTypeValidator.instance设置序列化时在 JSON 中嵌入类型信息，
        // 允许多态对象传输（父类接收、子类发送）、泛型对象传输（如 List<User>）、自定义复杂对象（非 final 类）都能保证精准反序列化
        // DefaultTyping.NON_FINAL 设置对非 final 修饰的类自动添加类型信息（final 类如 String、Integer 无需多态，因此不处理）
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance,
                ObjectMapper.DefaultTyping.NON_FINAL);

        Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter(objectMapper);

        // 添加设置消息的 MessageId，同时消费端存储 MessageId到数据库与新的消息MessageId比对做幂等性判断
        // 但是实际用到更多的还是根据业务实际判断来达到幂等性
        jackson2JsonMessageConverter.setCreateMessageIds(true);

        return new Jackson2JsonMessageConverter(objectMapper);
    }
}
~~~

序列化对象案例，如果生产者和消费者不在同一服务中则需要抽取成公共的依赖
~~~java
package com.entity;

import java.io.Serializable;

public class User implements Serializable {
    private String name;
    private Integer age;

    public User() {
    }

    public User(String name, Integer age) {
        this.name=name;
        this.age=age;
    }

    public String getName() {
        return name;
    }

    public Integer getAge() {
        return age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
~~~

4. 消息的生产发送
~~~java
import com.entity.User;
import lombok.RequiredArgsConstructor;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageProperties;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequiredArgsConstructor
public class VerifyController {

    static String queueName = "hello";

    // 广播类型的交换机
    static String fanoutExchange = "fanout_test";

    // 定向类型的交换机
    static String directExchange = "direct_test";

    // topic类型的交换机
    static String topicExchange = "topic_test";


    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Autowired
    private Jackson2JsonMessageConverter jsonMessageConverter;

    @GetMapping("/send/{message}")
    public String sendMessage(@PathVariable String message) {

        // 消息直接发送到队列，不通过交换机，不过一般生产者不直接发送消息到队列，而是发送消息到交换机，由交换机进行路由转发到队列
        //rabbitTemplate.convertAndSend(queueName, message);

        // 消息发送到广播类型的交换机，会传递给每个绑定该交换机的队列，也就是同一条消息传递到多个队列
        if (message.contains("hello1")) {
            rabbitTemplate.convertAndSend(fanoutExchange, null, message);
        }

        // 消息发送到定向类型的交换机，发送消息时指定RoutingKey，会将消息发送到BindingKey与RoutingKey一致的队列，可以根据消息的一些属性等进行判断决定发送到的队列
        if (message.contains("红")) {
            rabbitTemplate.convertAndSend(directExchange, "red", message);
        } else if (message.contains("蓝")) {
            rabbitTemplate.convertAndSend(directExchange, "blue", message);
        }

        // 消息发送到topic类型的交换机，发送消息时指定RoutingKey，会将消息发送到BindingKey与RoutingKey匹配的队列，可以根据消息的一些属性等进行判断决定发送到的队列
        rabbitTemplate.convertAndSend(topicExchange, "china.abc", message+"china.abc");
        rabbitTemplate.convertAndSend(topicExchange, "abc.weather", message+"abc.weather");
        rabbitTemplate.convertAndSend(topicExchange, "china.weather", message+"中国天气");


        // 对象序列化发送
        User user = new User(message, 13);
        MessageProperties props = new MessageProperties();
        props.setHeader("__TypeId__", "com.entity.User"); // 发送时在header中携带信息
        Message msg = jsonMessageConverter.toMessage(user, props);
        rabbitTemplate.convertAndSend("object_test", "User", msg);


        return "一切正常";
    }

}

~~~

5. 消息的接收消费
- 通过 @RabbitListener的bindings属性可以设置交换机和队列的绑定关系，同时交换机和队列不存在也会创建对应的交换机和队列
- 当然也可以通过手动创建队列、交换机和绑定关系的Bean,不过有些麻烦，没有消费者直接注解声明简便就不写对应例子了
~~~java
import com.entity.User;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Headers;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;

import java.util.Map;

@Component
public class Consumer {

    // 只声明了队列监听，没有绑定交换机，默认不会自动的创建队列
    @RabbitListener(queues = "hello")
    public void processHelloMessage(String content) {
        System.out.println("收到了hello队列的消息：" + content);
    }


    // 声明队列、交换机的绑定关系，并且设置队列持久化，交换机类型，队列和交换机如果不存在会自动的进行创建，无需手动进行创建
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "hello1", declare = "true"),
            exchange = @Exchange(name = "fanout_test", type = ExchangeTypes.FANOUT))
    )
    public void processHello1Message(String content) {
        System.out.println("收到了hello1队列的消息：" + content);
    }


    // direct 定向类型的交换机可以设置RoutingKey进行精确匹配
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "red_queue", declare = "true"),
            exchange = @Exchange(name = "direct_test", type = ExchangeTypes.DIRECT),
            key = {"red"}
    ))
    public void processRedQueueMessage(String content) {
        System.out.println("收到了红队的消息：" + content);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "blue_queue", declare = "true"),
            exchange = @Exchange(name = "direct_test", type = ExchangeTypes.DIRECT),
            key = {"blue"}
    ))
    public void processBlueQueueMessage(String content) {
        System.out.println("收到了蓝队的消息：" + content);
    }


    // topic类型的交换机可以设置RoutingKey进行模糊匹配
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "china_queue", declare = "true"),
            exchange = @Exchange(name = "topic_test", type = ExchangeTypes.TOPIC),
            key = {"china.#"}
    ))
    public void processChinaQueueMessage(String content) {
        System.out.println("收到了中国的消息：" + content);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "weather_queue", declare = "true"),
            exchange = @Exchange(name = "topic_test", type = ExchangeTypes.TOPIC),
            key = {"#.weather"}
    ))
    public void processWeatherQueueMessage(String content) {
        System.out.println("收到了天气的消息：" + content);
    }


    // 接收序列化对象消息
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "user_queue", declare = "true"),
            exchange = @Exchange(name = "object_test", type = ExchangeTypes.DIRECT),
            key = {"User"}
    ))
    public void processBlueQueueMessage(@Payload User user, @Headers Map<String, Object> headers, @Header(required = false) String error) {
        if (error != null) {
            // 处理异常消息
            String type = String.valueOf(headers.get("__TypeId__"));
            System.out.println("处理" + type + "对象异常：" + error);
            return;
        }
        System.out.println("User对象属性name=" + user.getName());
        System.out.println("User对象属性age=" + user.getAge());
    }

}
~~~


## 生产者保证消息不丢失的额外配置
1. 配置文件添加配置
- 设置重连机制避免网络波动
- 开启publisher-confirm-type确认ACK
- 开启publisher-returns获取发送失败的消息和详细原因（一般使用的比较少，因为有publisher-confirm-type确认ACK，但是如果消息无法路由到队列的消息会直接丢弃，但是ack是返回ok的，这种ack正常但是消息丢失的情况需要开启这个配置查看详细原因）
~~~yml
spring:
  rabbitmq:
    listener:
      connection-timeout: 1s      # 连接超时时长
    template:
      # 当网络不稳定的时候，利用重试机制可以有效提高消息发送的成功率。
      # 不过SpringAMQP提供的重试机制是阻塞式的重试，也就是说多次重试等待发送消息的过程中，当前线程是被阻塞的，会影响业务性能。
      # 如果对于业务性能有要求，建议禁用重试机制。如果一定要使用，请合理配置等待时长和重试次数，当然也可以考虑使用异步线程来执行发送消息的代码。
      retry:
        enabled: true                   # 开启超时重试
        initial-interval: 200ms         # 失败后的等待时间段
        multiplier: 1                   # 失败后下次的等待时长倍数，下次等待时长 = initial-interval * multiplier
        max-attempts: 3                 # 最大重试次数
    publisher-confirm-type: correlated  # 默认none:关闭confirm机制，simple:同步阻塞等待MQ的返回结果，correlated:MQ异步回调方式返回结果，通过回执消息可以进行判断ack是否ok
    publisher-returns: true             # 默认false关闭消息返回机制，无法路由到队列的消息直接丢弃（不会返回给生产者，但是ack是返回ok的）
~~~

2. 开启publisher-confirm-type确认ACK
~~~java
import lombok.RequiredArgsConstructor;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.concurrent.ListenableFutureCallback;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequiredArgsConstructor
public class VerifyController {

    // topic类型的交换机
    static String topicExchange = "topic_test";

    // 没有绑定队列的交换机
    static String noQueueExchange = "xxx";

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/send/{message}")
    public String sendMessage(@PathVariable String message) {
        
        //1.创建CorrelationData
        CorrelationData cd = new CorrelationData();
        //2.给Future添加ConfirmCallback
        cd.getFuture().addCallback(new ListenableFutureCallback<CorrelationData.Confirm>() {
            @Override
            public void onFailure(Throwable ex) {
                //2.1.Future发生异常时的处理逻辑，基本不会触发
                System.out.printf("handle message ack fail %s", ex.getMessage());
            }

            @Override
            public void onSuccess(CorrelationData.Confirm result) {
                //2.2.Future接收到回执的处理逻辑，参数中的result就是回执内容
                if (result.isAck()) {// result.isAck(),boolean类型,true代表ack回执,false 代表 nack回执
                    System.out.println("发送消息成功，收到 ack!");
                } else {
                    System.out.printf("发送消息失败，收到 nack，异常信息：%s", result.getReason());
                }

            }

        });

        // 必须设置 spring.rabbitmq.publisher-confirm-type: correlated 才能使用异步回调确认消息ack
        rabbitTemplate.convertAndSend(topicExchange, "china.weather", message, cd);
        // 消息发送到无队列绑定的交换机，会ack成功，但是消息丢失了必须使用设置 spring.rabbitmq.publisher-returns: true 全局设置开启publisher-returns的ReturnsCallback回调函数，注意这里的异步回调并不是ReturnsCallback的回调
        rabbitTemplate.convertAndSend(noQueueExchange, "china.weather", message, cd);

        return "一切正常";

    }
}
~~~


3. 全局设置开启publisher-returns的回调函数（一般使用的比较少，这个配置有一定的额外网络和系统开销。因为已经有publisher-confirm-type确认ACK了，但是如果消息无法路由到队列的消息会直接丢弃，但是ack是返回ok的，这种ack正常但是消息丢失的特殊情况需要开启这个配置查看详细原因）
~~~java
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class RabbitmqConfig implements ApplicationContextAware {

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        //获取RabbitTemplate
        RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
        // 必须设置 spring.rabbitmq.publisher-returns: true  开启消息返回机制才能确认
        // 设置ReturnCallback
        rabbitTemplate.setReturnsCallback(returned ->
                System.out.printf("消息发送失败，应答码%d，原因%s，交换机%s，路由键%s,消息%s %n",
                        returned.getReplyCode(), returned.getReplyText(), returned.getExchange(), returned.getRoutingKey(), returned.getMessage().toString())
        );
    }
}
~~~

## 消费者保证消息不丢失的额外配置
1. 配置文件添加配置
SpringAMQP已经实现了消息确认功能。并允许通过配置文件选择ACK处理方式，有三种方式:
- none：默认就是这个值，表示不处理。即消息投递给消费者后立刻ack，消息会立刻从MQ删除。非常不安全，不建议使用
- manual：手动模式。需要自己在业务代码中调用api，发送ack或reject，存在业务入侵，但更灵活
- auto：自动模式。SpringAMQP利用AOP对我们的消息处理逻辑做了环绕增强，当业务正常执行时则自动返回ack.当业务出现异常时，根据异常判断返回不同结果:
    - 如果是业务异常，会自动返回nack
    - 如果是消息处理或校验异常，自动返回reject
~~~yml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto            # 开启自动处理ACK，默认值none在处理消息的过程中出现异常会直接丢失消息
        # 消费者开启重试机制
        retry:
          enabled: true                   # 消费者开启超时重试
          initial-interval: 200ms         # 消费者失败后的等待时间段
          multiplier: 1                   # 消费者失败后下次的等待时长倍数，下次等待时长 = initial-interval * multiplier
          max-attempts: 3                 # 消费者最大重试次数
~~~

2. 设置消费者消息处理失败后的策略
在开启重试模式后，重试次数耗尽，如果消息依然失败，则需要有MessageRecoverer接口来处理，它包含三种不同的实现:
- RejectAndDontRequeueRecoverer：重试耗尽后，直接reject，丢弃消息。默认就是这种方式（一定不能使用这种，消息丢失不可取）
- ImmediateRequeueMessageRecoverer：重试耗尽后，返回nack，消息重新入队（不建议使用，因为重新入队还是大概率会失败）
- RepublishMessageRecoverer：重试耗尽后，将失败消息投递到指定的交换机（建议使用这种方式，重试多次依旧失败的消息需要人工处理）
~~~java
// 重试耗尽后，spring会自动的将消息投递到指定的交换机，可以在绑定该交换机的消息队列中看到具体的异常栈信息和该消息的原来所在的交换机名称、routingKey、以及消息内容等信息
@Bean
public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate) {
    // 设置交换机和routingKey
    // 对于原本不同业务的消息消费失败可以通过routingKey进行划分将失败的消息发送到不同的队列中
    return new RepublishMessageRecoverer(rabbitTemplate, "error.direct", "error");
}
~~~


























