# SpringBoot接入Kafka
1. 引入依赖
~~~
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
~~~

2. 配置文件添加配置
~~~yml
spring:
  kafka:
    bootstrap-servers: 192.168.31.209:9092,192.168.31.209:9093,192.168.31.209:9094
    consumer:
      group-id: my-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      enable-auto-commit: false  # 关闭自动提交
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    template:
      default-topic: test1
    listener:
      ack-mode: manual  # 开启手动确认
~~~

3. 接入生产者
~~~java
import lombok.RequiredArgsConstructor;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.util.concurrent.ListenableFutureCallback;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RequiredArgsConstructor
public class VerifyController {

    @Value("${spring.kafka.template.default-topic}")
    String topic;

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @GetMapping("/send/{message}")
    public String sendMessage(@PathVariable String message) {

        // 直接发送：在一些情况下不关心消息是否发送成功情况下使用
        //kafkaTemplate.send(topic, message);

        // 异步发送：设置回调函数
        kafkaTemplate.send(topic, message).addCallback(new ListenableFutureCallback<SendResult<String, String>>() {

            // 发送成功
            @Override
            public void onSuccess(SendResult<String, String> result) {
                RecordMetadata recordMetadata = result.getRecordMetadata();
                String topic = recordMetadata.topic();
                int partition = recordMetadata.partition();
                long offset = recordMetadata.offset();
                int i1 = recordMetadata.serializedKeySize();
                int i2 = recordMetadata.serializedValueSize();
                System.out.println("topic=" + topic + ", partition=" + partition + ", offset" + offset + ", serializedKeySize=" + i1 + ", serializedValueSize=" + i2);

            }

            // 发送失败
            @Override
            public void onFailure(Throwable ex) {
                System.err.println("发送失败: " + ex.getMessage());
                // 可做重试、告警、落库等
            }
        });

        return "一切正常";
    }

}
~~~

4. 接入消费者
~~~java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Component;

@Component
public class KafkaConsumer {

    @KafkaListener(
            topics = "${spring.kafka.template.default-topic}",
            groupId = "${spring.kafka.consumer.group-id}"
    )
    public void listen(String message, Acknowledgment ack) {
        // 处理消息
        System.out.println("Received message: " + message);
        ack.acknowledge();  // 手动提交偏移量
    }
}
~~~




