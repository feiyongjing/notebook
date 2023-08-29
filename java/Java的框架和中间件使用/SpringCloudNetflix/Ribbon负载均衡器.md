### Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。
通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。
Spring Cloud Ribbon虽然只是一个工具类框架，它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。
因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的
 

# Spring Cloud使用Ribbon 
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

~~~java
@Configuration
public class RestConfig {
    @LoadBalanced // 使用Ribbon实现负载均衡调用
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder){
        return builder.build();
    }
}
~~~

~~~java
@RestController
@RequestMapping("/parent_goods")
public class TestController {
    // 注意SPRING-CLOUD-GOODS是微服务程序的名字，而不是注册中心服务实例的id
    public static String GOODS_SERVICE_URL_2="http://SPRING-CLOUD-GOODS/goods/test?goodsId={goodsId}";

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/parent_test")
    private String getTest(@RequestParam("goodsId") String goodsId){
        Map<String,String> params=new HashMap<>();
        params.put("goodsId",goodsId);
        // 第一参数是微服务的url，第二参数是响应类型，第三参数是请求的实际参数
        ResponseEntity<String> entity = restTemplate.getForEntity(GOODS_SERVICE_URL_2, String.class,params);
        System.out.println(entity.getStatusCode());
        System.out.println(entity.getStatusCodeValue());
        System.out.println(entity.getHeaders());
        System.out.println(entity.getBody());
        return entity.getBody();
    }
}
~~~
 

### 注意上述使用是集成了注册中心的，Ribbon也可以不使用注册中心进行单点连接其他的微服务

# 几种负载均衡策略
- RoundRobinRule 轮询策略	默认值，启动的服务被循环访问
- RandomRule 随机选择	随机从服务器列表中选择一个访问
- BestAvailableRule 最大可用策略	先过滤出故障服务器，再选择一个当前并发请求数最小的服务
- WeightedResponseTimeRule 带有加权的轮询策略	对各个服务器响应时间进行加权处理，再采用轮询的方式获取相应的服务器
- AvailabilityFilteringRule 可用过滤策略	先过滤出故障的或并发请求大于阈值的服务实例，再以线性轮询的方式从过滤后的实例清单中选出一个
- ZoneAvoidanceRule 区域感知策略	先使用主过滤条件（区域负载器，选择最优区域）对所有实例过滤并返回过滤后的实例清单，依次使用次过滤条件列表中的过滤条件对主过滤条件的结果进行过滤，判断最小过滤数（默认1）和最小过滤百分比（默认0），最后对满足条件的服务器则使用RoundRobinRule(轮询方式)选择一个服务器实例	
- RetryRule 先轮询策略， 服务不可用就重试，先设定一个阈值时间段，如果在这个阈值时间段内重试不成功，则按照轮询分发给其他的服务


# 配置负载均衡策略

### 方式一：java的config进行设置全局的负载均衡策略
~~~java
    @Bean
    public RandomRule iRule(){
        return new RandomRule(); // 不同的策略返回不同的IRule
    }
~~~
 
### 方式二：application.properties或yml等配置文件设置单个微服务的负载均衡策略，SERVICE-TEST是微服务ID
~~~
SERVICE-TEST.ribbon.NFLoadBalancerRuleClassName = com.netflix.loadbalancer.RandomRule
~~~
 

### 方式三：也是设置单个微服务的负载均衡策略
新建一个config配置类，注意这个配置类不要在SpringBoot的组件扫描路径中如下
~~~java
@Configuration
public class MyConfiguration {
    @Bean
    public RandomRule iRule(){
        return new RandomRule(); // 不同的策略返回不同的IRule
    }
}
~~~

然后在启动类上添加
~~~java
@RibbonClients(value = {
        @RibbonClient(name = "stock-server",configuration = MyConfiguration.class),
        @RibbonClient(name = "这个name是指定微服务的名字",configuration = 这个配置类声明负载均衡策略)
})
~~~
 
