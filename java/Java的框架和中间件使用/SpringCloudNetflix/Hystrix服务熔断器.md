### Hystrix是服务熔断器，用于解决服务熔断降级和限流的情况。

## 服务熔断降级
- 主要是解决服务熔断、服务降级和限流的问题。
- 服务熔断主要用了解决服务雪崩，我们先来介绍一下服务雪崩的概念。
- 服务雪崩：假如有很多个服务都需要调用A服务，但A突然卡住了，响应很慢，在高并发的情况下，此时A服务就会持有了过量的资源，而导致其他服务的资源不足，从而影响其他服务的使用，甚至可能传播性地导致整个系统崩溃。
- 其实熔断，就好像我们生活中的跳闸一样， 比如说你的电路出故障了，为了防止出现大型事故 这里直接切断了你的电源以免意外继续发生, 把这个概念放在我们程序上也是如此， 当一个微服务调用多次出现问题时（默认是10秒内20次当然 这个也能配置），hystrix就会采取熔断机制，不再继续调用你的方法（会在默认5秒钟内和电器短路一样，5秒钟后会试探性的先关闭熔断机制，但是如果这时候再失败一次{之前是20次}那么又会重新进行熔断） 而是直接调用降级方法，这样就一定程度上避免了服务雪崩的问题。
- 降级是当我们的某个微服务响应时间过长，或者不可用了，讲白了也就是那个微服务调用不了了，我们不能吧错误信息返回出来，或者让他一直卡在那里，所以要在准备一个对应的策略（一个方法）当发生这种问题的时候我们直接调用这个方法来快速返回这个请求，不让他一直卡在那 。
- 限流就是限制你某个微服务的使用量（限制调用微服务可用线程数）
- 什么情况下会熔断降级调用降级方法？1、远程服务不可用。2、远程服务抛出异常。3、远程服务执行超时。

# 使用Hystrix

### 1. 添加Hystrix依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

### 2. 项目程序启动类上添加注解，多种注解选择一个：@EnableHystrix、@EnableCircuitBreaker、@SpringCloudApplication

### 3. 服务的调用者调用失败服务降级的处理

~~~java
@RestController
@RequestMapping("/parent_goods")
public class TestController {
    // 使用Feign调用
    @Autowired
    GoodsRemoteClient goodsRemoteClient;

    @HystrixCommand(fallbackMethod = "fallback")  // 声明当前的方法调用超时、异常会熔断降级失败后的降级方法
  //@HystrixCommand(fallbackMethod = "fallback",
  //                ignoreExceptions = Throwable.class) 出现指定的异常后不调用降级方法而是继续向外抛
    @GetMapping("/parent_test")
    private String getTest1(@RequestParam("goodsId") String goodsId){
        return goodsRemoteClient.getTest(goodsId);
    }
    
    // 服务不可用会调用这个方法处理失败
    // throwable参数是导致服务降级抛出异常（这个参数可加可不加，建议添加）
    public String fallback(String goodsId, Throwable throwable){
        return "调用服务时，服务熔断降级了";
    }
}
~~~

### 注意Hystrix默认是一秒内服务调用没有响应就会触发服务降级

# 修改Hystrix超时降级时间

1. 修改yml文件
~~~yml
hystrix:
  command:
    defult:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 5000 # 设置Hystrix的超时时间5000毫秒
~~~

2. 修改@HystrixCommand的属性
~~~java
@HystrixCommand(fallbackMethod = "timeout_handler", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")  // 设置超时时间1500毫秒
    })
~~~

注意服务降级实际的超时时间是Hystrix超时时间与Ribbon的ReadTimeout中时间短的生效
~~~
ribbon:
  ReadTimeout: 6000 # 服务提供者处理请求的超时时间
  ConnectTimeout: 3000 # Ribbon处理请求连接的超时时间
~~~

# Hystrix设置服务限流

### 1. 在Controller层的方法上添加注解属性threadPoolKey和threadPoolProperties

threadPoolKey 就是在线程池唯一标识， hystrix 会拿你这个标识去计数，看线程占用是否超过了， 超过了就会直接降级该次调用

比如， 这里coreSize给他值为2，maxQueueSize是1，那么假设你这个方法调用时间是3s执行完， 那么在3s内如果有超过3个请求进来的话， 超过的请求则全部降级

~~~java
@HystrixCommand(fallbackMethod = "fallback",
            threadPoolKey = "goods", // 设置线程池的名字
            threadPoolProperties = { 
                @HystrixProperty(name = "coreSize",value = "2"),         // 线程池的最大核心线程数
                @HystrixProperty(name = "maxQueueSize",value = "1")      // 线程池来不及处理任务时，阻塞队列堆积的最大任务数
            })
~~~

# 使用Feign整合Hystrix

### 1. feign 默认是支持hystrix的， 但是在Spring - cloud Dalston 版本之后就默认关闭了， 因为不一定业务需求要用的到，所以现在要使用首先得打开他，在yml文件加上如下配置:
~~~yml
feign:
  hystrix:
    enabled: true
~~~

### 2. 在Feign接口上的@FeignClient注解添加fallback属性声明降级方法所在的类
~~~java
@FeignClient(value = "SPRING-CLOUD-GOODS",               // 声明调用微服务的名字
             fallback = GoodsRemoteClientFallBack.class  // 声明服务降级方法所在的类
            ）
public interface GoodsRemoteClient {
    /**
     * 声明一个Feign的接口，它的实现是服务提供者的Controller层实现
     * 微服务的消费者调用当前接口就相当于调用服务提供者的Controller层
     * 注意这里使用的路径映射注解要与服务提供者的注解保证完全一致，
     * 即服务提供者使用@RequestMapping这里就使用@RequestMapping，其他的REST路径映射注解也一样，
     * 否则Feign在整合Hystrix或其他的东西时，服务消费者在调用服务提供者时会出现服务提供者是null出现空指针错误
     */
    @GetMapping("/test")
    String getTest(@RequestParam("goodsId")String goodsId);
}
~~~

### 3. 创建第二步中fallback属性声明降级方法所在的类
~~~java
@Component   // 注意这个类需要spring识别，所有是组件
public class GoodsRemoteClientFallBack implements GoodsRemoteClient{   // 继承Feign接口，实现的方法对应Controller路径映射方法的降级方法

    @Override
    public String getTest(String goodsId) {
        return "调用服务时，服务熔断降级了";
    }
}
~~~

### 如果需要在调用降级方法时获取异常，则不走上述的第二三步，按照以下步骤获取异常

### 1. 在Feign接口上的@FeignClient注解添加fallbackFactory属性声明工厂类，用于生产降级方法所在的类对象
~~~java
@FeignClient(value = "SPRING-CLOUD-GOODS", // 声明调用微服务的名字
             fallbackFactory = GoodsRemoteClientFallBackFactory.class  // 声明工厂类
)
public interface GoodsRemoteClient {
    /**
     * 声明一个Feign的接口，它的实现是服务提供者的Controller层实现
     * 微服务的消费者调用当前接口就相当于调用服务提供者的Controller层
     * 注意这里使用的路径映射注解要与服务提供者的注解保证完全一致，
     * 即服务提供者使用@RequestMapping这里就使用@RequestMapping，其他的REST路径映射注解也一样，
     * 否则Feign在整合Hystrix或其他的东西时，服务消费者在调用服务提供者时会出现服务提供者是null出现空指针错误
     */
    @GetMapping("/test")
    String getTest(@RequestParam("goodsId")String goodsId);
}
~~~

### 2. 创建第一步fallbackFactory属性声明声明的工厂类
~~~java
// 注意FallbackFactory这个类要引入 import feign.hystrix.FallbackFactory; 不要引错了，否则项目启动不了
@Component // 注意这个类需要spring识别，所有是组件
public class GoodsRemoteClientFallBackFactory implements FallbackFactory<GoodsRemoteClient> {
    @Override
    public GoodsRemoteClient create(Throwable cause) {  // 工厂方法能获取到异常
        return new GoodsRemoteClient(){   // 匿名内部类继承Feign接口，实现的方法对应Controller路径映射方法的降级方法
            @Override
            public String getTest(String goodsId) {  
                System.out.println("获取异常信息"+cause.getMessage());
                return null;
            }
        };
    }
}
~~~
 