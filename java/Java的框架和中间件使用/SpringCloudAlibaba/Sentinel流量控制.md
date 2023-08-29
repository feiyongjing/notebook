### Sentinel 是什么？
随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

# 代码配置流控规则和使用
### 1. 服务生产者项目添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
</dependency>
~~~
### 2. 设置流控规则和使用
~~~java
@RestController
@RequestMapping("/order")
public class OrderController {
    private static final String RESOURCE_NAME = "hello";
    
    // @PostConstruct声明方法是当前Controller创建的初始化方法
    // 配置流控规则
    @PostConstruct
    private static void initFlowRules() {
        // 定义一组流控规则
        List<FlowRule> flowRules = new ArrayList<>();
        // 设置具体流控规则
        FlowRule flowRule = new FlowRule();
        // 设置受保护的资源，一般是url的一部分或者是全部，即请求方法接收上的路径，不包含Controller类上的路径
        flowRule.setResource(RESOURCE_NAME);
        // 设置流控规则QPS
        flowRule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        // 设置受保护的资源阈值，设置为1,意味每秒受到保护的资源只能访问1次，超过的请求被限流或被降级
        flowRule.setCount(1);

        flowRules.add(flowRule);
        // 加载流控规则组
        FlowRuleManager.loadRules(flowRules);
    }

    @RequestMapping("/hello")
    public String hello() {
        Entry entry = null;
        try {
            // 针对资源进行限制
            entry = SphU.entry(RESOURCE_NAME);
            // 处理请求
            System.out.println("hello world");
            return "hello world";
        } catch (BlockException e1) {
            // 资源访问阻止，被限流降级或被熔断降级
            System.out.println("block!");
            return "被流控限流或被降级";
        } finally {
            if (entry != null) {
                entry.exit();
            }
        }
    }
}
~~~
--- 

### 注意上述的使用方式过于繁琐，对此可以使用AOP切面简化，简化流程如下

### 添加依赖(当然上述sentinel-core核心依赖不能少)
~~~xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
</dependency>
~~~

### 配置SentinelResourceAspect的Bean
~~~java
@Configuration
public class SentinelAspectConfiguration {
    @Bean
    public SentinelResourceAspect sentinelResourceAspect(){
        return new SentinelResourceAspect();
    }
}
~~~

### 设置流控规则和使用
~~~java
@RestController
@RequestMapping("/order")
public class OrderController {
    private static final String RESOURCE_NAME = "hello";
    
    // @PostConstruct声明方法是当前Controller创建的初始化方法
    // 配置流控规则
    @PostConstruct
    private static void initFlowRules() {
        // 定义一组流控规则
        List<FlowRule> flowRules = new ArrayList<>();
        // 设置具体流控规则
        FlowRule flowRule = new FlowRule();
        // 设置受保护的资源，一般是url的一部分或者是全部，即请求方法接收上的路径，不包含Controller类上的路径
        flowRule.setResource(RESOURCE_NAME);
        // 设置流控规则QPS
        flowRule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        // 设置受保护的资源阈值，设置为1,意味每秒受到保护的资源只能访问1次，超过的请求被限流或被降级
        flowRule.setCount(1);

        flowRules.add(flowRule);
        // 加载流控规则组
        FlowRuleManager.loadRules(flowRules);
    }

    // 使用@SentinelResource来替换原来的try catch，value属性指定限制的资源，
    // blockHandler属性指定当前类一个方法是限流降级或熔断降级方法
    // 如果不想在当前类中指定限流降级方法则使用blockHandlerClass在其他的类中指定限流降级或熔断降级方法，但是注意这种方式指定的降级方法必须是static修饰的
    // fallback属性指定当前类一个方法是异常降级方法
    // 如果不想在当前类中指定异常降级方法则使用fallbackClass在其他的类中指定降级方法，但是注意这种方式指定的降级方法必须是static修饰的
    // 还可以使用统一的BlockExceptionHandler指定异常降级的处理
    // 注意当限流降级或被熔断降级方法和异常降级方法都有指定时，如果触发限流则只会执行限流降级或被熔断降级方法，如果没有触发限流而是抛出异常则会执行异常降级方法
    // exceptionsToIgnore属性用于排除出现指定异常不进行处理
    @RequestMapping("/hello")
    @SentinelResource(value = RESOURCE_NAME, 
                  //  blockHandlerClass = HelloHandler.class,
                      blockHandler = "blockExceptionHandler",
                  //  fallback = "exceptionFallbackHandler",
                  //  fallbackClass = ExceptionFallback.class
                  //  exceptionsToIgnore = IOException.class,
                    )
    public String hello1() {
        // 处理请求
        System.out.println("hello world");
        return "hello world";
    }

    // 限流降级或熔断降级方法，注意一定的public修饰，返回值要与原方法保持一致，可以添加原方法的参数
    // 必须在参数列表最后添加一个BlockException类型的参数用于获取具体的异常信息
    public String blockExceptionHandler(BlockException e1) {
        // 资源访问阻止，被限流或被降级
        System.out.println("block!");
        return "被流控限流或被降级";
    }
    
    // 异常降级方法，注意一定的public修饰，返回值要与原方法保持一致，可以添加原方法的参数
    // 必须在参数列表最后添加一个Throwable类型的参数用于获取具体的异常信息
    public String exceptionFallbackHandler(Throwable throwable) {
        // 资源访问出现异常阻止，被异常降级
        System.out.println("block!");
        return "被异常降级";
    }
}
~~~
 

 

# 代码配置降级规则和使用

### 1. 服务生产者项目添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
</dependency>
~~~

### 2. 配置SentinelResourceAspect的Bean
~~~java
@Configuration
public class SentinelAspectConfiguration {
    @Bean
    public SentinelResourceAspect sentinelResourceAspect(){
        return new SentinelResourceAspect();
    }
}
~~~
 
### 3. 设置降级规则和使用
~~~java
@RestController
@RequestMapping("/order")
public class OrderController {
    private static final String RESOURCE_NAME = "hello";
    
    @PostConstruct
    private static void initDegradeRules() {
        // 定义一组降级规则
        List<DegradeRule> degradeRules = new ArrayList<>();
        // 设置具体降级规则
        DegradeRule degradeRule = new DegradeRule();
        // 设置受保护的资源，一般是url的一部分或者是全部，即请求方法接收上的路径，不包含Controller类上的路径
        degradeRule.setResource(RESOURCE_NAME);

        // 设置规则侧率：当前选择的是异常数
        degradeRule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT);
        // 触发熔断的异常数: 2
        degradeRule.setCount(2);
        // 触发熔断的最小请求数: 2
        degradeRule.setMinRequestAmount(2);
        // 统计时长: 默认单位是ms，现在设置为1分钟
        degradeRule.setStatIntervalMs(60*1000);
        // 熔断持续时长：10s
        degradeRule.setTimeWindow(10);

        // 上述配置详细解释
        // 在统计时长(一分钟)内请求达到触发熔断的最小请求数并且请求出现异常的次数达到触发熔断的异常数会进入半开状态
        // 在半开状态下的第一次请求如果还是出现异常，则会直接进入持续的熔断降级
        // 熔断的持续时长设置10s，在熔断的持续时长内的请求都会熔断降级，熔断的持续时长结束后回到半开状态

        degradeRules.add(degradeRule);
        // 加载降级规则组
        DegradeRuleManager.loadRules(degradeRules);
    }

    @RequestMapping("/hello")
    @SentinelResource(value = RESOURCE_NAME,
            blockHandler = "blockExceptionHandler",
            fallback = "exceptionFallbackHandler"
    )
    public String hello1() {
        // 处理请求
        System.out.println("hello world");
        throw new RuntimeException("抛出异常");
    }
    
    // 限流降级或熔断降级方法，注意一定的public修饰，返回值要与原方法保持一致，可以添加原方法的参数
    // 必须在参数列表最后添加一个BlockException类型的参数用于获取具体的异常信息
    public String blockExceptionHandler(BlockException e1) {
        // 资源访问阻止，被限流或被降级
        System.out.println("block!");
        return "被流控限流或被降级";
    }
    
    // 异常降级方法，注意一定的public修饰，返回值要与原方法保持一致，可以添加原方法的参数
    // 必须在参数列表最后添加一个Throwable类型的参数用于获取具体的异常信息
    public String exceptionFallbackHandler(Throwable throwable) {
        // 资源访问出现异常阻止，被异常降级
        System.out.println("block!");
        return "被异常降级";
    }
}
~~~
 

# Sentinel的统一异常降级处理BlockExceptionHandler接口

当出现多个请求需要进行统一异常降级处理时去除@SentinelResource的fallback属性 和 fallbackClass 属性，然后创建一个组件实现BlockExceptionHandler接口，例如
~~~java
@Component
public class MyBlockExceptionHandler implements BlockExceptionHandler {

    Logger log= LoggerFactory.getLogger(this.getClass());

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        // 记录资源规则的详细信息
        log.info("BlockExceptionHandler BlockException=========="+ e.getRule());
        Result<?> r=null;
        if(e instanceof FlowException){
            r=Result.error(100,"接口降级了");
        }else if(e instanceof DegradeException){
            r=Result.error(101,"服务降级了");
        }else if(e instanceof ParamFlowException){
            r=Result.error(102,"热点参数限流了");
        }else if(e instanceof SystemBlockException){
            r=Result.error(103,"触发系统保护规则了");
        }else if(e instanceof AuthorityException){
            r=Result.error(104,"服务授权不通过");
        }
        response.setStatus(500);
        response.setCharacterEncoding("utf-8");
        response.setContentType(MediaType.APPLICATION_JSON);
        new ObjectMapper().writeValue(response.getWriter(),r);

    }
    
    @Getter
    @Setter
    public static class Result<T> {
        private Integer code;
        private String msg;
        private T data;
        public Result(Integer code, String msg) {
            this.code = code;
            this.msg = msg;
        }
        public static Result<?> error(Integer code, String msg){
            return new Result<>(code,msg);
        }
    }
}
~~~

# Sentinel控制台启动与客户端连接
 参考：https://github.com/alibaba/Sentinel/wiki/%E6%8E%A7%E5%88%B6%E5%8F%B0 下载sentinel的jar包

启动控制台命令
~~~shell
java -Dserver.port=8088 -Dsentinel.dashboard.auth.username=sentinel -Dsentinel.dashboard.auth.password=123456 -jar sentinel-dashboard-1.8.0.jar
~~~

## 客户端连接

### 1. 客户端项目添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
</dependency>
~~~

### 2. 客户端项目启动时添加JVM参数
~~~shell
-Dcsp.sentinel.dashboard.server=127.0.0.1:8088 # 指定sentinel控制台启动服务器的ip和端口
~~~

### 3. 启动客户端项目，访问下其中的接口，再去查看Sentinel控制台就可以看到相关的设置了
---

# Sentinel与Spring Cloud Alibaba整合
### 1. 与单独的使用Sentinel不同，只需要添加Sentinel的启动器依赖就可以了，不需要添加多余的依赖 
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
~~~
### 2. 操作Sentinel控制台设置不同的规则
---

# 控制台不同规则的介绍
### 1. 流控规则：选择对应的资源在新建流控规则时，设置资源名、阈值类型、阈值数、流控模式、流控效果
- 资源名指的是url路径的一部分或者是全部

- 阈值类型分为QPS和线程数
  - QPS是指在一秒内的请求数，选择QPS阈值类型则一秒最多处理阈值数之内请求，一秒内超过阈值数的请求会进行限流降级
  - 线程数是指在一秒内最多只有对应阈值数的线程正常处理请求，一秒内多余的请求会进行限流降级

- 流控模式分为直接、关联、链路
  - 默认是直接，即上述设置的阈值类型和阈值数是对当前资源请求的限制
  - 关联模式选择后需要设置关联资源名，即上述设置的阈值类型和阈值数是对关联资源请求的限制，当请求超对关联资源的阈值后，会对当前资源限流 
  - 关联模式适合用于读写请求时，减少读写之间竞争资源，即写关联读，即读请求超过阈值后会对写限流，让写减少资源让给读，反之亦然
  - 链路模式选择后需要设置入口资源名，是指服务提供者被多个调用者调用时，入口资源选择其中的一个或者多个进行限流，其他的不限流
---
注意使用链路模式需要添加配置
~~~yml
spring:
  cloud:
    sentinel:
      web-context-unify: false    # 默认是true将调用链路收敛
~~~

- 流控效果分为快速失败、warm up（预热）、排队等待
  - 默认是快速失败，即超出阈值的请求直接失败，进行降级处理
  - warm up 预热是指面对急剧上升的洪峰流量请求，设置预热时长在预热时长内慢慢的让服务器处理效率达到阈值，当然处理不过来的请求还是会直接失败降级处理，防止突然的流量压垮服务器，可以解决缓存穿透，缺点是洪峰流量请求持续时间短，洪峰流量请求时间过后没多久再次迎来洪峰流量请求，导致进行多余的预热使本来可以正常处理的请求失败处理，对服务器造成多余的浪费
  - 排队等待是指超出阈值的请求进行排队等待，设置超时时长即排队等待的时间，如果超出阈值的请求排队等待时间超过超时时长则直接失败，进行降级处理
  - 注意上述时间设置的单位都是毫秒
---

### 2. 降级规则：选择对应的资源在新建降级规则时，设置资源名、熔断策略、熔断时长、最小请求数
- 资源名指的是url路径的一部分或者是全部
- 熔断策略分为慢调用比例、异常比例、异常数
  - 默认是慢调用比例需要设置最大RT、比例阈值，当请求的响应时间超过最大RT，这个请求就是慢调用。比例阈值是是指一定时间之内慢调用处理请求的数量除以总的请求数，当总的请求数量需要达到或者超过设置的最小请求数时计算比例阈值，如果超过了设置的比例阈值则会进行按照设置的熔断时长持续的熔断，熔断时长过后会进入半开状态，半开状态下只要发生一次慢调用就会再次进行持续的熔断

  - 异常比例需要设置比例阈值，比例阈值是是指一定时间之内请求处理出现异常的数量除以总的请求数，当总的请求数量需要达到或者超过设置的最小请求数时计算比例阈值，如果超过了设置的比例阈值则会进行按照设置的熔断时长持续的熔断，熔断时长过后会进入半开状态，半开状态下只要发生一次慢调用就会再次进行持续的熔断

  - 异常数需要设置异常的数量，在一定时间之内请求处理出现异常的数量达到设置的异常数量，并且总的请求数达到或者超过设置的最小请求数，则会进行按照设置的熔断时长持续的熔断，熔断时长过后会进入半开状态，半开状态下只要发生一次慢调用就会再次进行持续的熔断
---

### 3. 热点规则：选择对应的资源在新建热点规则时，设置资源名、参数索引、单机阈值、统计窗口时长和一组或多组的参数类型、参数值、限流阈值
- 资源名指的是url路径的一部分或者是全部

- 参数索引是请求方法参数列表的索引从0开始，例如第一个参数、第二个参数之类的

- 单机阈值和统计窗口时长是对QPS限流的设置，即QPS等于单机阈值除以统计窗口时长，注意这个单机阈值是针对该资源名所有的请求限流设置

- 参数类型、参数值、限流阈值是针对该资源所有请求中附带参数符合条件的请求进行特定限流阈值设置的限流，即请求的参数列表中在上述设置的参数索引所在位置的参数值与一组参数匹配则这个请求就是在访问热点数据，会进行特定限流阈值设置的限流

- 注意使用热点规则不能使用统一的异常降级处理必须使用@SentinelResource​的属性指定资源名和限流降级方法

- 热点规则适用于请求带有指定实参的请求限流或者是对其放开流量，比如对热点商品的访问量大，可以针对携带这些请求携带这些固定热点商品ID参数的接口进行限流
---

### 4. 系统规则：对服务器系统设置一些限制指标，当触发这些限制指标后会请求直接进行系统降级处理
- 系统保护规则设置阈值类型和阈值
- 阈值类型有LOAD、RT、线程数、入口QPS、CPU使用率

- cpu使用率是指当服务器的系统cpu达到设置的阈值（阈值在0到1之间）后，其他的请求全部进行系统降级处理

- 入口QPS是针对服务器提供的所有接口计算平均的阈值是否超过设置的阈值，超过阈值的请求进行系统降级处理

 

# Sentinel与Nacos整合实现控制台配置的规则持久化

### 1. 服务提供者项目添加添加依赖
~~~xml
<strong><dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency></strong>
~~~
 
### 2. 在Nacos的控制台手动创建一个配置，例如这里创建了一个data Id是order-sentinel的json配置文件，其中配置了流控规则，内容如下，但是其他的规则有其他的配置
~~~
[
    {
        "resource": "addOrder",    # 这个对应控制台流控规则的资源名
        "grade": "1",              # 这个对应控制台流控规则的阈值类型，1对应的是QPS
        "count": "1",              # 这个对应控制台流控规则的单机阈值，即每秒多处理请求数量
        "limitApp": "default",     # 这个对应控制台流控规则的针对来源，默认是default
        "strategy": "0",           # 这个对应控制台流控规则的流控模式，0对应的是直接
        "controlBehavior": "0"     # 这个对应控制台流控规则的流控效果，0对应的是快速失败
    }
]
~~~

### 3. 服务提供者项目添加配置（可以在配置中心配置）
~~~yml
spring:
  cloud:
    sentinel:
      datasource:
        - nacos:
            server-addr: localhost:8849,localhost:8850,localhost:8851 # 连接的注册中心地址和端口
            username: nacos # 连接注册中心地址登录用户名
            password: nacos # 连接注册中心地址登录密码
            data-id: order-sentinel # 第二步在Nacos创建的配置文件data Id
            data-type: json         # 第二步在Nacos创建的配置文件格式
            group-id: order         # 第二步在Nacos创建的配置文件group
            namespace: 774b7a5e-df22-4173-9c39-45fd0f9b7adc  # 第二步在Nacos创建的配置文件所在的命名空间ID
            rule-type: flow         # 第二步在Nacos创建的配置文件内容是哪种Sentinel规则，如 flow、degrade、param-flow、gw-flow 等。
~~~

### 4. 使用@SentinelResource设置资源名和降级方法等，注意资源名要与第二步在Nacos创建的配置文件内容中的resource一致

### 5. 启动服务提供者项目，访问其中的接口，在Sentinel控制台可以看见设置的对应规则

 

# Sentinel与openFeign的整合，和Hystrix与openFeign的整合基本一致

### 注意如果使用openFeign，则需要服务的调用者与提供者在nacos注册中心服务所注册的服务命名空间和组名都保持一致

### 下面是整合的例子，如果需要在调用降级方法时获取异常，请参考Hystrix的笔记

1. 服务调用者项目在yml文件加上如下配置
~~~yml
feign:
  sentinel:
    enabled: true # 开启feign对sentinel的整合
~~~

2. 在Feign接口上的@FeignClient注解添加fallback属性声明降级方法所在的类
~~~java
@FeignClient(value = "stock-server",
        fallback = StockRemoteClientFallBack.class
)
public interface StockFeignService {
    @RequestMapping("/reduct")
    String reduct();
}
~~~

3. 创建第二步中fallback属性声明降级方法所在的类，注意这是一个组件类
~~~java
@Component
public class StockRemoteClientFallBack implements StockFeignService {
    @Override
    public String reduct() {
        return "调用服务时，服务熔断降级了";
    }
}
~~~
 
