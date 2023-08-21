1. 引入依赖
~~~xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.16.7</version>
</dependency>
~~~

2. Spring配置文件添加连接Redis信息
~~~yml
redis:
    host: 172.16.104.91
    port: 6379
~~~

3. 配置RedissonClient的Bean
~~~java
@Configuration
public class RedissonConfig {
    @Value("${redis.host}")
    private String host;
    @Value("${redis.port}")
    private Integer port;
    @Bean
    public RedissonClient redissonClient(){
        Config config = new Config();                           // Redisson 配置
        String url = "redis://" + host + ":" + port;            // 组装url
        config.useSingleServer()                                // 使用单机Redis服务连接
              .setAddress(url);                                 // 添加服务连接地址
        RedissonClient redisson = Redisson.create(config);      // 创建Redisson客户端
        return client;
    }
}
~~~
 

4. 使用可重入锁Reentrant Lock  或公平锁Fair Lock

~~~java
public void testReentrantLock(RedissonClient redisson){   // 在service中使用@autoWired注入

        try{
            RLock lock = redisson.getLock("anyLock"); //根据键key获取可重入锁
            RLock fairLock = redisson.getFairLock("anyLock"); //根据键key获取公平锁
            
            // 上锁，当前线程持有锁加一，如果拿不到锁则当前线程进入等待状态
            //lock.lock();

            // 上锁之后，支持过期解锁功能,10秒钟以后自动解锁, 无需调用unlock方法手动解锁
            //lock.lock(10, TimeUnit.SECONDS);

            // 3秒的时间尝试上锁，成功则立即返回true,否则立即返回false,当前线程上锁成功以后10秒自动解锁
            boolean res = lock.tryLock(3, 10, TimeUnit.SECONDS);
            if(res){    
                // 上锁成功的业务代码
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
             //lock锁是否是当前线程持有，持有则释放锁
             if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }

    }
~~~
 

5. 使用联锁MultiLock和红锁RedLock
~~~java
public void testMultiLock(RedissonClient redisson1,
                              RedissonClient redisson2, RedissonClient redisson3){

        RLock lock1 = redisson1.getLock("lock1");
        RLock lock2 = redisson2.getLock("lock2");
        RLock lock3 = redisson3.getLock("lock3");

        RedissonMultiLock MultiLock = new RedissonMultiLock(lock1, lock2, lock3);  //使用联锁关联3个锁
        RedissonRedLock RedLock = new RedissonRedLock(lock1, lock2, lock3);      //使用红锁关联3个锁  
        try {
            // 联锁同时加锁：lock1 lock2 lock3, 所有的锁都上锁成功才算成功。其他联锁上锁同理
            MultiLock.lock();

            //红锁同时上锁：lock1 lock2 lock3, 大部分的锁都上锁成功才算成功。其他红锁上锁同理
            RedLock.lock();

            // 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
            boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
            if(res){    
                // 上锁成功的业务代码
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }
~~~
