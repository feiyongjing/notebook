# Kafka客户端基本使用
1. 引入依赖
~~~xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.8.0</version>
</dependency>
~~~

2. 生产者常见配置和生产者Demo
生产者常见配置如下（源码文档请看 org.apache.kafka.clients.producer.ProducerConfig 类）
~~~
bootstrap.servers 
    list。
    用于建立与kafka集群的连接，这个list仅仅影响用于初始化的hosts，来发现全部的servers。
    格式：host1:port1,host2:port2,…，数量尽量不止一个，以防其中一个down了。
 
acks 
    字符串, 默认值是1。
    Server完成 producer request 前需要确认的数量。
    acks=0时，producer不会等待确认，直接添加到socket等待发送，性能好可靠性低
    acks=1时，等待leader写到local log就行，性能和可靠性均衡
    acks=all或acks=-1时，等待isr中所有副本确认，性能差但是可靠性高，但是可以对服务端设置 min.insync.replicas，也就是设置Kafka 要求至少有多少个 ISR（同步副本） 成功写入消息，建议设置为集群节点数的多半数量
    如果你开启了幂等性（enable.idempotence=true），acks 必须设为 all
    如果产生配置冲突（如幂等开启但 acks=1），Kafka 会自动关闭幂等性。
    （注意：确认都是 broker 接收到消息放入内存就直接返回确认，不是需要等待数据写入磁盘后才返回确认，这也是kafka快的原因）
 
buffer.memory
    long, 默认值33554432。
    Producer可以用来缓存数据的内存大小。该值实际为RecordAccumulator类中的BufferPool，即Producer所管理的最大内存。
    如果数据产生速度大于向broker发送的速度，producer会阻塞max.block.ms，超时则抛出异常。
 
compression.type
    字符串，默认值none。
    Producer用于压缩数据的压缩类型，取值：none, gzip, snappy, or lz4。但是注意也需要提前在服务端设置compression.type为对应的压缩算法
    
batch.size
    int，默认值16384。
    Producer可以将发往同一个Partition的数据做成一个Produce Request发送请求，即Batch批处理，以减少请求次数，该值即为每次批处理的大小。
    另外每个Request请求包含多个Batch，每个Batch对应一个Partition，且一个Request发送的目的Broker均为这些partition的leader副本。
    若将该值设为0，则不会进行批处理。

interceptor.classes
    string
    自定义拦截器全限定类名，多个拦截器用逗号分隔，拦截器的调用顺序会按配置顺序调用，默认情况下，没有拦截器
    自定义拦截器需要实现org.apache.kafka.clients.producer.ProducerInterceptor
 
linger.ms
    long, 默认值0。
    Producer默认会把两次发送时间间隔内收集到的所有Requests进行一次聚合，然后再发送，以此提高吞吐量，而linger.ms则更进一步，这个参数为每次发送增加一些delay，以此来聚合更多的Message。
    官网解释翻译：producer会将request传输之间到达的所有records聚合到一个批请求。通常这个值发生在欠负载情况下，record到达速度快于发送。但是在某些场景下，
    client即使在正常负载下也期望减少请求数量。这个设置就是如此，通过人工添加少量时延，而不是立马发送一个record，producer会等待所给的时延，以让其他records发送出去，
    这样就会被聚合在一起。这个类似于TCP的Nagle算法。该设置给了batch的时延上限：当我们获得一个partition的batch.size大小的records，就会立即发送出去，而不管该设置；
    但是如果对于这个partition没有累积到足够的record，会linger指定的时间等待更多的records出现。该设置的默认值为0(无时延)。例如，设置linger.ms=5，会减少request发送的数量，
    但是在无负载下会增加5ms的发送时延。
 
max.request.size
    int, 默认值1048576。
    请求的最大字节数。这也是对最大消息大小的有效限制。注意：server具有自己对消息大小的限制，这些大小和这个设置不同。此项设置将会限制producer每次批量发送请求的数目，以防发出巨量的请求。
 
receive.buffer.bytes
    int, 默认值32768。
    TCP的接收缓存 SO_RCVBUF 空间大小，用于读取数据。
 
request.timeout.ms
    int，默认值30000。
    client等待请求响应的最大时间,如果在这个时间内没有收到响应，客户端将重发请求，超过重试次数发送失败。
 
 
send.buffer.bytes
    int，默认值131072。
    TCP的发送缓存 SO_SNDBUF 空间大小，用于发送数据。
    
timeout.ms
    int，默认值30000。
    指定server等待来自followers的确认的最大时间，根据acks的设置，超时则返回error。
 
max.in.flight.requests.per.connection
    int，默认值5。
    在block前一个connection上允许最大未确认的requests数量。当设为1时，即是消息保证有序模式，
    注意：这里的消息保证有序是指对于单个Partition的消息有顺序，因此若要保证全局消息有序，可以只使用一个Partition，当然也会降低性能
 
metadata.fetch.timeout.ms
    long，默认值60000。
    在第一次将数据发送到某topic时，需先fetch该topic的metadata，得知哪些服务器持有该topic的partition，该值为最长获取metadata时间。
 
reconnect.backoff.ms
    long，默认值50.
    连接失败时，当我们重新连接时的等待时间。
 
retry.backoff.ms
    long， 默认值100。
    在重试发送失败的request前的等待时间，防止若目的Broker完全挂掉的情况下Producer一直陷入死循环发送，折中的方法。
 
metrics.sample.window.ms
    long, 默认值3000
    metrics系统维护可配置的样本数量，在一个可修正的window size
 
metrics.num.samples
    int, 默认值2
    用于维护metrics的样本数
 
metric.reporters
    数组[]
    类的列表，用于衡量指标。实现MetricReporter接口
 
metrics.log.level
    metrics日志记录级别，默认info
 
metadata.max.age.ms
    long, 默认值300000
    强制刷新metadata的周期，即使leader没有变化
 
security.protocol
    默认值PLAINTEXT
    与broker会话协议，取值：LAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL
 
partitioner.class
    默认值class org.apache.kafka.clients.producer.internals.DefaultPartitioner
    分区类，实现Partitioner接口
    
max.block.ms
    long，默认值60000
    控制block的时长，当buffer空间不够或者metadata丢失时产生block
 
connections.max.idle.ms
    long，默认值540000
    设置多久之后关闭空闲连接
 
client.id
    字符串，默认值""
    当向server发出请求时，这个字符串会发送给server，目的是能够追踪请求源
 
retries
    int，默认值0
    发生错误时，重传次数。当开启重传时，需要将`max.in.flight.requests.per.connection`设置为1，否则可能导致失序
    默认最大重试次数是int最大值，但是实际重试会被 delivery.timeout.ms (2 分钟) 限制，2 分钟内还发不出去就放弃
 
key.serializer
    key 序列化方式，类型为class，需实现Serializer interface， 如：org.apache.kafka.common.serialization.StringSerializer
 
value.serializer
    value 序列化方式，类型为class，需实现Serializer interface，如org.apache.kafka.common.serialization.StringSerializer
 
enable.idempotence
    true为开启幂等性
 
transaction.timeout.ms 
    事务超时时间，默认60000，单位ms
 
transactional.id 
    设置事务id，必须唯一
~~~

生产者Demo如下
~~~java
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

/**
 * kafka生产者
 */
public class Producer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        Properties properties = new Properties();
        // Kafka的生产者有三个必选的属性：
        // bootstrap.servers，指定broker的地址列表。
        // key.serializer必须是一个实现org.apache.kafka.common.serialization.Serializer接口的类，将key序列化成字节数组。注意：key.serializer必须被设置，即使消息中没有指定key。
        // value.serializer，将value序列化成字节数组。
        properties.put("bootstrap.servers", "localhost:9092");
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        // 参考上面的生产者配置
        properties.put("acks", "all");
        properties.put("retries", 0);
        properties.put("batch.size", 16384);
        properties.put("linger.ms", 1);
        properties.put("buffer.memory", 33554432);

        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<String, String>(properties);
        for (int i = 1; i <= 600; i++) {

            // ProducerRecord的构造参数必须提供topic和value，其他构造函数还提供了可选的partition、key参数
            // partition参数可以设置消息发送到指定的分区
            // 当不指定key参数时，key值默认为null。消息的key有两个重要的作用：
            // 1、提供描述消息的额外信息。
            // 2、用来决定消息写入到哪个分区，所有“具有相同key的消息”会分配到同一个分区中。
            //    如果key为null，而且使用的是默认的分配器，那么会随机选择一个分区分配。
            //    如果key不为null，而且使用的是默认的分配器，那么生产者会对key进行哈希，并根据结果将消息分配到特定的分区。
            // 注意：在计算消息与分区的映射关系时，使用的是全部的分区数而不仅仅是可用的分区数。这也意味着，如果某个分区不可用，而消息刚好被分配到该分区，那么将会写入失败。
            // 另外，如果需要增加额外的分区，那么消息与分区的映射关系将会发生改变（每条历史消息也会根据key计算哈希重新分配分区），因此，尽量避免这种情况。
            ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>("test20200519", "Kafka_Products", "message" + i);

            // 同步发送：在一些情况下不关心消息是否发送成功，则不需要从Future对象获取发送结果，future.get获取结果会阻塞的
            // Future future = kafkaProducer.send(producerRecord);
            // future.get();

            // 异步发送：设置回调函数
            kafkaProducer.send(producerRecord, new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    if (e != null) { // 如果Kafka返回一个错误，onCompletion方法抛出一个non null异常。
                        e.printStackTrace(); // 对异常进行一些处理，这里只是简单打印出来
                    } else {
                        // 可以获取发送的topic、partition、offset、消息key和value大小
                        String topic = recordMetadata.topic();
                        int partition = recordMetadata.partition();
                        long offset = recordMetadata.offset();
                        int i1 = recordMetadata.serializedKeySize();
                        int i2 = recordMetadata.serializedValueSize();
                        System.out.println("topic=" + topic + ", partition=" + partition + ", offset" + offset + ", serializedKeySize=" + i1 + ", serializedValueSize=" + i2);
                    }
                }
            });


            System.out.println("message" + i);
        }
        kafkaProducer.close();
    }
}
~~~

3. 消费者常见配置和消费者Demo
消费者常见配置如下（源码文档请看 org.apache.kafka.clients.consumer.ConsumerConfig 类）
~~~
group.id
    字符串
    消费者所属消费组的唯一标识。
 
max.poll.records 
    int，默认值500
    一次拉取请求的最大消息数。
 
max.poll.interval.ms 
    int， 默认值300000
    指定拉取消息线程最长空闲时间。
 
session.timeout.ms 
    int， 默认值10000
    检测消费者是否失效的超时时间。
 
heartbeat.interval.ms 
    int， 默认值3000
    消费者心跳时间，默认3000ms。
 
bootstrap.servers 
    连接集群broker地址。
 
enable.auto.commit 
     boolean， 默认值true
    是否开启自动提交消费offset位移的功能。
 
auto.commit.interval.ms 
    int，默认值5000
    自动提交消费位移的时间间隔。
 
partition.assignment.strategy 
    range 或 roundrobin，默认的值是range。
    消费者的分区配置策略。
 
auto.offset.reset 
    字符串，默认值"latest"
    如果分区没有初始偏移量，或者当前偏移量服务器上不存在时，将使用的偏移量设置。earliest从头开始消费，latest从最近的开始消费，none抛出异常。
 
fetch.min.bytes 
    int，默认值1
    消费者客户端一次请求从Kafka拉取消息的最小数据量，如果Kafka返回的数据量小于该值，会一直等待，直到满足这个配置大小，默认1b。
 
fetch.max.bytes 
    int，默认值50*1024*1024，即50MB
    消费者客户端一次请求从Kafka拉取消息的最大数据量。
 
fetch.max.wait.ms 
    int，默认500。
    从Kafka拉取消息时，在不满足fetch.min.bytes条件时，等待的最大时间。
 
metadata.max.age.ms 
    强制刷新元数据时间，毫秒，默认300000，5分钟。
 
max.partition.fetch.bytes 
    int，默认1*1024*1024，即1MB
    设置从每个分区里返回给消费者的最大数据量，区别于fetch.max.bytes。
 
send.buffer.bytes
    int，默认128*1024
    Socket发送缓冲区大小，默认128kb,-1将使用操作系统的设置。
 
receive.buffer.bytes 
    Socket发送缓冲区大小，默认64kb,-1将使用操作系统的设置。
 
client.id 
    消费者客户端的id。
 
reconnect.backoff.ms 
    连接失败后，尝试连接Kafka的时间间隔，默认50ms。
 
reconnect.backoff.max.ms 
    尝试连接到Kafka，生产者客户端等待的最大时间，默认1000ms。
 
retry.backoff.ms 
    消息发送失败重试时间间隔，默认100ms。
 
metrics.sample.window.ms 
    样本计算时间窗口，默认30000ms。
 
metrics.num.samples 
    用于维护metrics的样本数量，默认2。
 
metrics.log.level 
    metrics日志记录级别，默认info。
 
metric.reporters 
    类的列表，用于衡量指标，默认空list。
 
check.crcs 
    boolean，默认true
    自动检查CRC32记录的消耗。
 
key.deserializer 
    key反序列化方式。
 
value.deserializer 
    value反序列化方式。
 
connections.max.idle.ms 
    设置多久之后关闭空闲连接，默认540000ms。
 
request.timeout.ms 
    客户端将等待请求的响应的最大时间,如果在这个时间内没有收到响应，客户端将重发请求，超过重试次数将抛异常，默认30000ms。
 
default.api.timeout.ms 
    设置消费者api超时时间，默认60000ms。
 
interceptor.classes 
    自定义拦截器。
 
exclude.internal.topics 
    内部的主题:一consumer_offsets 和一transaction_state。该参数用来指定 Kafka 中的内部主题是否可以向消费者公开，默认值为 true。如果设置为 true，那么只能使用 subscribe(Collection)的方式而不能使用 subscribe(Pattern)的方式来订阅内部主题，设置为 false 则没有这个限制。
 
isolation.level 
    用来配置消费者的事务隔离级别。如果设置为“read committed”，那么消费者就会忽略事务未提交的消息，即只能消 费到 LSO (LastStableOffset)的位置，默认情况下为 “read_uncommitted”，即可以消 费到 HW (High Watermark)处的位置。
~~~

消费者Demo如下
~~~java
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

/**
 * kafka消费者
 */
public class Consumer {

    public static void main(String[] args) {
        // Kafka的消费者有三个必选的属性：
        // bootstrap.servers，指定broker的地址列表。
        // key.deserializer必须是一个实现org.apache.kafka.common.serialization.Deserializer接口的类，将key反序列化成字符串。注意：key.deserializer必须被设置，即使消息中没有指定key。
        // value.deserializer，将value反序列化成字符串。
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        // 参考上面的消费者配置
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("auto.offset.reset","earliest");

        // KafkaConsumer类不是线程安全的
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("test20200519")); // 订阅topic
        try {
            while (true) {
                // 拉取消息
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("partition=%d, offset = %d, key = %s, value = %s%n",record.partition(), record.offset(), record.key(), record.value());
                }

                // 同步和异步提交消息，提交完消息在kafka的partition中offset变更表示这批消息消费成功
                // 当然上面配置了 enable.auto.commit = true 进行自动提交，可以不用手动提交了
                // 但是还是有些问题，提交完消息在kafka的partition中offset变更到底推进了多少客户端无法得知（服务端宕机），所以一些大厂会使用额外的redis等存储在消费消息之后手动的记录该消费者组在topic中每个partition的offset，这样客户端可以在下一次从kafka拉取的消息与 redis中存储的每个topic下partition的offset 进行比对可以明确的新拉取消息是否已经消费处理过了，以达到避免因kafka中topic下partition的offset意外丢失或者错误导致的重复消费消息
                //consumer.commitSync();
                //consumer.commitAsync();
            }
        } finally {
            consumer.close();
        }
    }
}

~~~


4. 生产者配置拦截器
- 生产者添加自定义拦截器（生产者调用顺序 ：拦截器->序列化器->分区器）
~~~java
// 添加自定义拦截器，多个拦截器用逗号分隔，拦截器的调用顺序会按配置顺序调用
properties.put("interceptor.classes", ProducerInterceptor.class.getName());
~~~

- 自定义拦截器实现
~~~java
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

import java.util.Map;

public class ProducerInterceptor implements org.apache.kafka.clients.producer.ProducerInterceptor<String, String> {
    private volatile long sendSuccess = 0;
    private volatile long sendFailure = 0;

    /**
     * 消息发送前触发，如下是进行修改消息
     */
    @Override
    public ProducerRecord onSend(ProducerRecord<String, String> producerRecord) {

        String modifieldValue = "prefix-" + producerRecord.value();
        return new ProducerRecord<String, String>(producerRecord.topic(),
                producerRecord.partition(),
                producerRecord.timestamp(),
                producerRecord.key(),
                modifieldValue,
                producerRecord.headers());
    }

    /**
     * 收到服务端响应时触发，如下是打印消息发送成功还是失败
     */
    @Override
    public void onAcknowledgement(RecordMetadata recordMetadata, Exception e) {
        if (e == null) {
            sendSuccess++;
            System.out.println("发送成功的消息:" + sendSuccess);
        } else {
            sendFailure++;
            System.out.println("发送失败的消息:" + sendFailure);
        }
    }

    /**
     * 连接关闭时触发
     */
    @Override
    public void close() {
        System.out.println("发送成功率：" + (double) sendSuccess / (sendSuccess + sendFailure));
    }

    /**
     * 发送消息前整理配置项，可以打印配置方便查看
     */
    @Override
    public void configure(Map<String, ?> map) {
        System.out.println("=====config start======");
        for (Map.Entry<String, ?> entry : map.entrySet()) {
            System.out.println("entry.key:" + entry.getKey() + " === entry.value: " + entry.getValue());
        }
        System.out.println("=====config end=======");
    }
}
~~~


5. 生产者和消费者配置序列化器
如果Kafka客户端提供的几种序列化器都无法满足业务，则可以使用Avro/JSON/Thrift/ProtoBuf和Protostuff等通用的序列化工具来实现，或者使用自定义类型的序列化器来实现，序列化器对Kafka的传输效率提高很明显

- 假设有一个User类作为消息传递的value，需要自定义User类的序列化器
~~~java
public class User {
    private String name;
    private int age = -1;
    public String getName() {
        return name;
    }
    public User setName(String name) {
        this.name = name;
        return this;
    }
    public int getAge() {
        return age;
    }
    public User setAge(int age) {
        this.age = age;return this;
    }
}
~~~

- 定义发送消息时的User类序列化器
~~~java
import org.apache.kafka.common.serialization.Serializer;

import java.io.UnsupportedEncodingException;
import java.nio.ByteBuffer;
import java.util.Map;

//自定义序列化器
public class UserSerializer implements Serializer<User> {

    @Override
    public void configure(Map<String, ?> map, boolean b) {

    }

    @Override
    public byte[] serialize(String s, User user) {
        if (user == null) {
            return null;
        }
        byte[] name;
        int age = user.getAge();

        try {
            if (user.getName() != null) {
                name = user.getName().getBytes("UTF-8");
            } else {
                name = new byte[0];
            }
            // 数组总共的长度：一个int统计name字节数（int占4个位）+ name字节数（name属性占用的位）+ age（age属性是int类型占4个位）
            // 为啥要这样设计序列化？
            // 当这样的byte[]数组被消费者接收到了之后，可以明确的知道前4个位表示的是name字节数，可以快速的确定name属性有多少位找到name属性，name后面的4位是age，然后在消费者端如果出现粘包的情况下可以快速的判断包是否完整
            // 相比User对象转Json字符串然后转byte[]数组要更小，传输效率更加高
            ByteBuffer byteBuffer = ByteBuffer.allocate(4+4+name.length);
            //name字节数
            byteBuffer.putInt(name.length);
            //放name字节数组
            byteBuffer.put(name);
            //放age,age本身就是int类型的
            byteBuffer.putInt(age);
            return byteBuffer.array();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return new byte[0];
    }

    @Override
    public void close() {

    }

}
~~~

- 定义消费接收消息时的User类反序列化器
~~~java
import org.apache.kafka.common.serialization.Deserializer;

import java.nio.ByteBuffer;
import java.util.Map;

public class UserDeserializer implements Deserializer<User> {
    @Override
    public void configure(Map<String, ?> map, boolean b) {

    }
    @Override
    public User deserialize(String s, byte[] bytes) {
        ByteBuffer byteBuffer = ByteBuffer.wrap(bytes);
        int nameLength = byteBuffer.getInt();
        byte name[] = new byte[nameLength];
        byteBuffer.get(name,0,nameLength);
        int age = byteBuffer.getInt();
        return new User().setAge(age).setName(new String(name));
    }
    @Override
    public void close() {
    }
}
~~~

- 生产者添加自定义User类序列化器（生产者调用顺序 ：拦截器->序列化器->分区器）
~~~java
properties.put("value.serializer", UserSerializer.class.getName());
~~~

- 消费者添加自定义User类反序列化器
~~~java
properties.put("value.deserializer", UserDeserializer.class.getName());
~~~

6. 生产者配置分区路由器
如果ProducerRecord消息中指定了partition字段那么就不需要分区器的作用了，因为partition代表的就是所要发往的分区号。
但是消息中没有指定partition字段，那么就需要依赖分区器，根据key这个字段来计算partition的值。分区器的作用就是为消息分配分区。

Kafka中提供的默认分区器是DefaultPartitioner，
默认分区器将消息先滞留缓存，直到滞留缓存至少有 BATCH_SIZE_CONFIG （客户端批处理的大小（默认是16KB），没有到达该批处理大小就等待 LINGER_MS_CONFIG （默认是0） 毫秒） 字节的数据为止，然后按照下面的策略进行批量发送消息到分区
如果key不为null,那么默认的分区器会对key进行哈希（采用MurmurHash2算法，具备高运算性能及低碰撞率），最终根据得到的哈希值来计算分区号,拥有相同的key的消息会被写入同一个分区
如果key为null,那么将会随机选择一个分区将上述准备批处理的消息发送到这个分区
在不改变topic中分区的情况下，key与分区之间的映射可以保持不变。不过，一旦topic中增加了新的分区，会对所以消息重新进行

org.apache.kafka.clients.producer.RoundRobinPartitioner 分区器才是每条消息轮询选择分区

- 生产者添加自定义分区器（生产者调用顺序 ：拦截器->序列化器->分区器）
~~~java
properties.put("partitioner.class", MyPartitioner.class.getName());
~~~

- 实现生产者的自定义分区器
~~~java
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;

import java.util.List;
import java.util.Map;

public class MyPartitioner implements Partitioner {

    public void configure(Map<String, ?> configs) {}

    /**
     * 返回选择的 partition 分区号
     * @param topic
     * @param key
     * @param keyBytes 序列化的key
     * @param value
     * @param valueBytes 序列化的value
     * @param cluster 集群的元信息
     */
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        // 这里直接返回的第一个partition分区号
        return availablePartitions.get(0).partition();

    }

    public void close() {}

}
~~~

7. 消费者选择分区消费策略
- org.apache.kafka.clients.consumer.RangeAssignor 按 Topic 独立分配：它是逐个 Topic 进行分配的，而不是全局考虑。对于每个 Topic，将分区按序号排序，消费者按 ID 排序，然后将分区划分为连续的区间分配给消费者。例如有一个topic有9个分区、3个消费者，1号消费者分配0、1、2分区，2号消费者分配3、4、5分区，3号消费者分配6、7、8分区

- org.apache.kafka.clients.consumer.RoundRobinAssignor 全局轮询，将所有 Topic 的所有分区合并成一个列表，排序后，像发牌一样轮询分给每个消费者。但是前提是消费者组内所有消费者必须订阅完全相同的 Topic，如果组内消费者订阅的 Topic 不同，会导致分配混乱且负载不均

- org.apache.kafka.clients.consumer.StickyAssignor 尽可能保持分区分配不变（粘性），同时保证最终分配是均衡的。满足均衡性：最终每个消费者分到的分区数量尽可能一致。粘性：重平衡时，尽量保留消费者当前已有的分区，只移动最少的分区来达到均衡

- org.apache.kafka.clients.consumer.CooperativeStickyAssignor核心思想： 在 StickyAssignor 的基础上，实现了 协作式重平衡    
传统重平衡（Eager）：重平衡开始时，所有消费者立即放弃所有分区，导致消费短暂停止（Stop-The-World），然后重新分配。
协作式重平衡（Cooperative）：
第一轮：告知消费者哪些分区必须放弃，哪些可以保留。
消费者继续处理保留的分区，只停止并提交放弃的分区。
第二轮：将放弃的分区分配给新消费者。
全程无全量停止，消费几乎不中断。

Kafka 3.8.0 的官方文档，默认的分区分配策略是 RangeAssignor 和 CooperativeStickyAssignor 的组合
默认行为：在这个组合下，实际生效的是 RangeAssignor。因为消费者会优先使用列表中第一个被所有成员都支持的策略。
但是建议在生产环境中显式配置，只使用 CooperativeStickyAssignor 在重平衡时性能更好、更稳定

8. 发送应答机制（面试八股文重点）
主要就是客户端的 ACKS_CONFIG 配置 默认值是1
- acks=0时，客户端不会等待确认请求的响应，直接添加到网络socket等待发送，性能最高，可靠性最低
- acks=1时，等待服务端 leader写到local log就行，不等待 Follower 同步，比 acks=0 稍慢，并且有一定可靠性：只要 Leader 不宕机，消息就安全
- acks=all或acks=-1时，等待isr（In-Sync Replicas，同步中的副本） 都写入成功后，才返回确认。性能最差，但是有最强可靠性：只要还有一个 ISR 副本存活，消息就不会丢失
（注意：确认都是 broker 接收到消息放入内存就直接返回确认，不是需要等待数据写入磁盘后才返回确认，这也是kafka快的原因）

当然acks=all或acks=-1时也可以对服务端所有节点进行调优
~~~
# min.insync.replicas表示：当生产者设置 acks=all 时，Kafka 要求至少有多少个 ISR（同步副本） 成功写入消息，才会向生产者返回 “成功”
# 建议设置为集群节点数量的多半，既保证了可靠性，性能也得到了提升
min.insync.replicas = 2
~~~

如果你开启了幂等性（enable.idempotence=true），acks 必须设为 all
如果产生配置冲突（如幂等开启但 acks=1），Kafka 会自动关闭幂等性。

9. Kafka 幂等性（面试八股文重点）
实现基于 PID（Producer ID）+ 序列号（Sequence Number）。具体流程如下：
- 初始化 PID：生产者启动时，会向服务端申请一个唯一的 PID。
- 维护序列号：生产者为每个 Topic 分区 维护一个从 0 开始递增的 序列号。
- 发送携带标识：每条消息都会携带 PID + 序列号。
- 服务端去重：服务端会缓存每个 PID + 分区 最近的序列号。
                如果收到的消息序列号 比缓存的SN大 → 认为是新消息写入（正常是比缓存的SN大1，如果超过就说明有消息丢失了），更新缓存。
                如果收到的消息序列号 等于或小于缓存的 → 认为是重复消息，直接丢弃，返回 ACK。
- 客户端保证消息不丢失：生产者消息被追加到 RecordAccumulator 时，幂等性会做一个关键检查
                正常情况：发送 Seq 0，成功。下一个发送 Seq 1，直接放行。
                异常情况（Seq 0 发送失败）：生产者准备发送 Seq 1、Seq 2。
                内部检查发现：Seq 0 还未成功。
                自动触发阻塞：将 Seq 1、Seq 2 的请求暂存在队列中，不发送到网络。
                直到 Seq 0 重试成功，队列才会自动释放，按顺序发送 Seq 1、Seq 2。
                

~~~plaintext
[Producer]                 [Broker]
   (内存)                    (内存 + Log)
     │                         │
     │  1. 申请 PID             │
     │────────────────────────>│
     │                         │
     │  2. 返回 PID (1001)      │
     │<────────────────────────│
     │                         │
     │  3. 发消息(PID=1001, Seq=0)│
     │────────────────────────>│───┐
     │                         │   │ 查内存缓存
     │                         │<──┘
     │                         │
     │                         │ 4. 写入Log，更新内存(PID=1001, SN=0)
     │                         │
     │  5. 发消息(PID=1001, Seq=0)│  <-- 重试/重复
     │────────────────────────>│
     │                         │───┐
     │                         │   │ 发现 Seq <= SN
     │  6. 直接返回 ACK (丢弃)  │<──┘
     │<────────────────────────│
~~~

  
开启幂等性（enable.idempotence=true）必须满足以下三个条件，否则会报错 ConfigException：
- acks=all（必须）：必须确保所有 ISR 副本都确认写入，才能保证 PID 和序列号不丢失。
- retries > 0（必须）：幂等性是为了解决重试带来的问题，如果不重试，就没有重复的风险。
- max.in.flight.requests.per.connection <= 5（必须）原因：为了保证消息顺序。afka 保证在这个限制内，即使并发请求，也能保证消息的顺序性和幂等性


10. Kafka事务
~~~java
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.ExecutionException;

/**
 * kafka生产者
 */
public class Producer {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //配置
        Properties properties = new Properties();
        //连接参数
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
        //序列化
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        //优化参数 可选
        //缓冲器大小 32M
        properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 32 * 1024 * 1024);
        //批次大小
        properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16 * 1024);
        //Linger.ms
        properties.put(ProducerConfig.LINGER_MS_CONFIG, 5);
        //压缩
        //properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");
        //acks
        properties.put(ProducerConfig.ACKS_CONFIG, "all");
        //重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 10);

        //指定事务ID
        properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional_id_01");
        properties.put("enable.idempotence", "true");

        //创建生产者
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);

        //事务消息 初始化
        kafkaProducer.initTransactions();
        //开始事务
        kafkaProducer.beginTransaction();
        try {
            for (int i = 1; i <= 5; i++) {
                kafkaProducer.send(new ProducerRecord<String, String>("first", "Transactions")).get();
                if(i==3){
                    // 假设出现意外手动回滚事务                    
                    kafkaProducer.abortTransaction();
                }
            }
            //提交事务
            kafkaProducer.commitTransaction();
        } catch (Exception e) {
            //回滚事务
            kafkaProducer.abortTransaction();
        } finally {
            //关闭资源
            kafkaProducer.close();
        }
    }

}
~~~










