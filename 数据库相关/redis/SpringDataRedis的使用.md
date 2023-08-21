以下内容参考自：https://www.cnblogs.com/lenve/p/10965667.html#:~:text=%E5%9C%A8%20Spring%20Boot%20%E4%B8%AD%EF%BC%8C%E9%BB%98%E8%AE%A4%E9%9B%86%E6%88%90%E7%9A%84%20Redis%20%E5%B0%B1%E6%98%AF%20Spring%20Data,Spring%20Data%20Redis%20%E9%92%88%E5%AF%B9%20Redis%20%E6%8F%90%E4%BE%9B%E4%BA%86%E9%9D%9E%E5%B8%B8%E6%96%B9%E4%BE%BF%E7%9A%84%E6%93%8D%E4%BD%9C%E6%A8%A1%E6%9D%BF%20RedisTemplate%20%E3%80%82%EF%BF%BD

1. 引入依赖 注意commons是用于管理Redis集群的
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
~~~
 
2. 在application.properties 中设置Redis连接的属性

~~~yml
spring.redis.database=0                        # Redis数据库索引（默认为0）
spring.redis.password=123                      # Redis服务器连接密码（默认为空）
spring.redis.port=6379                         # Redis服务器连接端口
spring.redis.host=192.168.66.128               # Redis服务器地址
spring.redis.lettuce.pool.min-idle=5           # 连接池中的最小空闲连接
spring.redis.lettuce.pool.max-idle=10          # 连接池中的最大空闲连接
spring.redis.lettuce.pool.max-active=8         # 连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=1ms         # 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.shutdown-timeout=100ms    # 连接超时时间（毫秒）
~~~

3. RedisAutoConfiguration类是默认的配置类  
默认提供了RedisTemplate 和 StringRedisTemplate 两个Bean，其中 StringRedisTemplate 是 RedisTemplate 的子类，两个的方法基本一致，不同之处主要体现在操作的数据类型不同，RedisTemplate 中的两个泛型都是 Object ，意味者存储的 key 和 value 都可以是一个对象，而 StringRedisTemplate 的 两个泛型都是 String ，意味者 StringRedisTemplate 的 key 和 value 都只能是字符串。如果开发者没有提供相关的 Bean ，这两个配置就会生效，否则不会生效。

也可以自己手动的配置RedisTemplate 的Bean 例如
~~~java
@Configuration
public class RedisConfig {
    @Bean
    RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }
}
~~~
 
具体的从Redis中存取、删除数据需要操作RedisTemplate类 这个Bean的API