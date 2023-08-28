### SpringCloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

SpringCloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 2.0之前的非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。

Spring Cloud Gateway 的目标，不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。

提前声明：Spring Cloud Gateway 底层使用了高性能的通信框架Netty。

### 几个重要概念
- Filter（过滤器）：和Zuul的过滤器在概念上类似，可以使用它拦截和修改请求，并且对上游的响应，进行二次处理。过滤器为org.springframework.cloud.gateway.filter.GatewayFilter类的实例。
- Route（路由）：网关配置的基本组成模块，和Zuul的路由配置模块类似。一个Route模块由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配，目标URI会被访问。
- Predicate（断言）：这是一个 Java 8 的 Predicate，可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。断言的输入类型是一个 ServerWebExchange

上述参考 1：https://blog.csdn.net/rain_web/article/details/102469745 
上述参考2：https://www.cnblogs.com/crazymakercircle/p/11704077.html 

# Gateway使用
---
## Gateway单独使用（不与注册中心集成）
1. ### 搭建一个Spring Boot应用并引入依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
~~~
---

2. ### 添加配置
~~~yml
spring:
  application:
    name: gateway-server
  cloud:
    gateway:
      routes:                 # 设置路由规则组，即数组的一个元素匹配一个服务的路由过滤规则
        - id: order-routes    # 路由的唯一标识
          uri: http://localhost:8081   # 需要路由的服务器ip和端口
          predicates:                  # 断言路由的匹配规则
            - Path=/order-server/**    # 路径符合添加的url请求会进行路由，现在设置的是匹配前缀路径是/order-server/的url
          filters:                     # 设置过滤规则
            - StripPrefix=1            # 对请求的url去除最前面一层目录后进行服务转发
server:
  port: 80
~~~
---

## Gateway与Nacos注册中心集成使用
1. ### 搭建一个Spring Boot应用并引入依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
~~~
---
2. ### 添加配置
~~~yml
spring:
  application:
    name: gateway-server
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8849 # 连接的注册中心地址和端口
        username: nacos # 连接注册中心地址登录用户名
        password: nacos # 连接注册中心地址登录密码
        namespace: 774b7a5e-df22-4173-9c39-45fd0f9b7adc
    gateway:
      routes:                 # 设置路由规则组，即数组的一个元素匹配一个服务的路由过滤规则
        - id: order-routes    # 路由的唯一标识
          uri: lb://order-server       # lb:// 是指使用nacos的本地负载均衡策略 order-server是注册中心中注册服务的名称
          predicates:                  # 断言路由的匹配规则
            - Path=/order-server/**    # 路径符合添加的url请求会进行路由，现在设置的是匹配前缀路径是/order-server/的url
          filters:                     # 设置过滤规则
            - StripPrefix=1            # 对请求的url去除最前面一层目录后进行服务转发
#      discovery:
#        locator:
#          enabled: true      # 使用约定大于配置，即默认使用Nacos注册的服务名发现服务，无需配置路由规则
server:
  port: 80
~~~
---
 
# Predicate（断言）相关配置
---
### 参考：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories 

根据情况选择其中的添加配置
~~~yml
spring:
  cloud:
    gateway:
      routes:                 
        - predicates:                  # 断言路由的匹配规则
            - After=2017-01-20T17:42:47.789-07:00[America/Denver] # 路由匹配服务器系统指定时间之后的请求
            - Before=2017-01-20T17:42:47.789-07:00[America/Denver] # 路由匹配服务器系统指定时间之前的请求
            - Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver] # 路由匹配服务器系统指定时间段的请求
            - Cookie=chocolate, ch.p # 路由匹配带有cookie名为chocolate，值与ch.p正则表达式匹配的请求
            - Header=X-Request-Id, \d+ # 路由匹配请求头Header中带有名为X-Request-Id，值与\d+正则表达式匹配的请求
            - Host=**.somehost.org,**.anotherhost.org # 路由请求的Host值匹配正则表达式的请求，可以设置多个正则表达式，使用逗号分隔
            - Method=GET,POST # 路由匹配方法的请求
            - Path=/red/{segment},/blue/{segment} # 路由url匹配的请求，{segment}指路径变量
            - Query=green # 路由请求带有指定表单参数的请求，例子是匹配包含green参数请求
            - Query=red, gree. # 路由请求带有指定表单参数并且值于正则表达式匹配的请求，例子是匹配包含green参数并且值于 gree. 匹配的请求
            - RemoteAddr=192.168.1.1/24 # 路由指定服务器发起的请求，例子是匹配192.168.1.1发起的请求，24是子网掩码可以不写
            - Weight=group1, 8 # 按照分组和值路由，每个组包含100%的流量请求，值是路由当前组的多少流量，例子是路由group1组中约 80% 的流量请求
~~~
---

# 自定义断言工厂
参考QueryRoutePredicateFactory﻿​设置的自定义的请求参数断言工厂，与其功能一致

### 注意规则
1. 这是一个spring的组件
2. 这个自定义断言工厂以RoutePredicateFactory结尾
3. 包含一个内部类存放配置属性实际设置值，并继承AbstractRoutePredicateFactory泛型是其中内部类
4. 与默认的断言工厂使用一致，都要在配置文件中设置具体的配置属性

~~~yml
spring:
  cloud:
    gateway:
      routes:                 
        - predicates:                  # 断言路由的匹配规则
            - CheckAuth=red, gree. # 与默认的Query的功能一致，CheckAuth是自定义断言工厂类的前缀
~~~
---

~~~java
@Component
public class CheckAuthRoutePredicateFactory extends AbstractRoutePredicateFactory<CheckAuthRoutePredicateFactory.Config> {

    // 构造器默认使用父类的
    public CheckAuthRoutePredicateFactory() {
        super(CheckAuthRoutePredicateFactory.Config.class);
    }

    // 简化配置，注意配置属性值的实际位置
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("param", "value");
    }

    @Override
    public Predicate<ServerWebExchange> apply(CheckAuthRoutePredicateFactory.Config config) {
        return new GatewayPredicate() {
            // 路由断言，true路由成功，false路由失败
            @Override
            public boolean test(ServerWebExchange exchange) {
                if (!StringUtils.hasText(config.value)) {
                    // check existence of header
                    return exchange.getRequest().getQueryParams()
                            .containsKey(config.param);
                }

                List<String> values = exchange.getRequest().getQueryParams()
                        .get(config.param);
                if (values == null) {
                    return false;
                }
                for (String value : values) {
                    if (value != null && value.matches(config.value)) {
                        return true;
                    }
                }
                return false;
            }

            @Override
            public String toString() {
                return String.format("CheckAuth: param1=%s value=%s", config.getParam1(),
                        config.getValue());
            }
        };
    }

    // 存放配置属性实际设置值的类
    @Validated
    public static class Config {

        @NotEmpty
        private String param;

        private String value;

        public String getParam1() {
            return param;
        }

        public Config setParam(String param) {
            this.param = param;
            return this;
        }

        public String getValue() {
            return value;
        }

        public Config setValue(String value) {
            this.value = value;
            return this;
        }
        

    }

}
~~~
---
 

# Filter（过滤器）相关配置
中文参考：https://www.jianshu.com/p/58267466251e 
官方参考：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories



###自定义过滤器工厂
分两种过滤器：局部过滤器和全局过滤器，局部过滤器针对一个路由规则过滤处理，全局过滤器针对所有路由请求过滤处理

## 局部过滤器使用
局部过滤器注意规则
1. 这是一个spring的组件
2. 这个自定义断言工厂以GatewayFilterFactory﻿​结尾
3. 包含一个内部类存放配置属性实际设置值，并继承AbstractGatewayFilterFactory﻿​泛型是其中内部类
4. 与默认的过滤器工厂使用一致，都要在配置文件中设置具体的配置属性

~~~yml
spring:
  cloud:
    gateway:
      routes:                 # 设置路由规则组，即数组的一个元素匹配一个服务的路由过滤规则
        - filters:                     # 设置过滤规则
            - CheckAuth=/mypath        # 与默认的PrefixPath的功能一致，CheckAuth是自定义过滤器工厂类的前缀 
~~~
---

~~~java
@Component
public class CheckAuthGatewayFilterFactory extends AbstractGatewayFilterFactory<CheckAuthGatewayFilterFactory.Config> {

    public static final String PREFIX_KEY = "prefix";

    private static final Log log = LogFactory
            .getLog(PrefixPathGatewayFilterFactory.class);

    // 构造器默认使用父类的
    public CheckAuthGatewayFilterFactory() {
        super(CheckAuthGatewayFilterFactory.Config.class);
    }

    // 简化配置，注意配置属性值的实际位置
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList(PREFIX_KEY);
    }

    @Override
    public GatewayFilter apply(CheckAuthGatewayFilterFactory.Config config) {
        return new GatewayFilter() {
            // 请求进行过滤处理
            @Override
            public Mono<Void> filter(ServerWebExchange exchange,
                                     GatewayFilterChain chain) {
                boolean alreadyPrefixed = exchange
                        .getAttributeOrDefault(GATEWAY_ALREADY_PREFIXED_ATTR, false);
                if (alreadyPrefixed) {
                    return chain.filter(exchange);
                }
                exchange.getAttributes().put(GATEWAY_ALREADY_PREFIXED_ATTR, true);

                ServerHttpRequest req = exchange.getRequest();
                addOriginalRequestUrl(exchange, req.getURI());
                String newPath = config.prefix + req.getURI().getRawPath();

                ServerHttpRequest request = req.mutate().path(newPath).build();

                exchange.getAttributes().put(GATEWAY_REQUEST_URL_ATTR, request.getURI());

                if (log.isTraceEnabled()) {
                    log.trace("Prefixed URI with: " + config.prefix + " -> "
                            + request.getURI());
                }

                return chain.filter(exchange.mutate().request(request).build());
            }

            @Override
            public String toString() {
                return filterToStringCreator(CheckAuthGatewayFilterFactory.this)
                        .append("prefix", config.getPrefix()).toString();
            }
        };
    }

    // 存放配置属性实际设置值的类
    public static class Config {

        private String prefix;

        public String getPrefix() {
            return prefix;
        }

        public void setPrefix(String prefix) {
            this.prefix = prefix;
        }

    }
}
~~~
---
 

## 全局过滤器使用
~~~java
@Component
public class LogFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        System.out.println("打印请求url: "+exchange.getRequest().getPath().value());
        return chain.filter(exchange);
    }
}
~~~
---
 

## 开启网关服务的日志
注意是在添加JVM启动参数，而不是Spring Boot的配置文件
~~~
-Dreactor.netty.http.server.accsssLogEnabled=true﻿​
~~~
---

# 跨域设置
Gateway默认是不允许跨域的，通过配置可以开启跨域请求转发
方式一：网关服务项目配置文件添加配置
~~~yml
spring:
  cloud:
    gateway:
      globalcors:   # 跨域配置
        cors-configurations:
          '[/**]':  # 允许跨域访问的资源
            allowedOrigins: "*"  # 跨域允许的来源
            allowedMethods:      # 跨域允许的方法
              - GET
              - POST
~~~
---
方式二：网关服务项目通过Java代码方式配置
~~~java
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;

@Configuration
public class CorsConfig {
    @Bean
    public CorsWebFilter corsFilter(){
        CorsConfiguration config=new CorsConfiguration();
        config.addAllowedMethod("*");   // 跨域允许的方法
        config.addAllowedOrigin("*");   // 跨域允许的来源
        config.addAllowedHeader("*");   // 跨域允许的请求头
        
        UrlBasedCorsConfigurationSource source=new UrlBasedCorsConfigurationSource(); // 注意引入的类
        source.registerCorsConfiguration("/**",config); // 允许跨域访问的资源
        return new CorsWebFilter(source);
    }
}
~~~
---
 

# Gateway与Sentinel整合
1. ### 网关服务项目添加依赖
~~~xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>
~~~
---
2. ### 网关服务项目配置文件添加配置
~~~yml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8088 # sentinel启动服务器的ip和端口
~~~
---

3. ### 重启gateway网关服务，登录sentinel控制台就可以配置流控规则了
- API管理中可以创建多个API组，一个API组即一组url路径
- 流控规则中API类型有Route ID和API分组
- Route ID是根据每个路由的ID来设置流控规则
- API分组是对一个API组中的所有url设置流控规则

 
网关服务在Sentinel控制台设置流控规则与一般的流控不同的地方
- 多了针对请求属性设置，QPS下多了间隔时间设置、Burst size设置

针对请求属性设置是指对请求的请求头、url、Cookie 等进行断言，通过断言的请求才会计算在QPS中进行流控
QPS下的间隔时间设置是指QPS等于阈值除以间隔时间，即原来间隔时间默认的一秒
QPS下的Burst size设置是指实际QPS间隔时间内可以超出阈值的请求数，即原来Burst size默认是0


# Gateway设置限流熔断处理请求返回
~~~java
@Configuration
public class GatewayConfig {
    Logger log= LoggerFactory.getLogger(this.getClass());
    @PostConstruct
    public void init(){
        BlockRequestHandler blockRequestHandler=new BlockRequestHandler() {
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable e) {
                HashMap<String,String> map=new HashMap<>();
                map.put("code", HttpStatus.TOO_MANY_REQUESTS.toString());
                log.info("异常信息："+e);
                String message="";
                if(e instanceof FlowException){
                    message="接口降级了";
                }else if(e instanceof DegradeException){
                    message="服务降级了";
                }else if(e instanceof ParamFlowException){
                    message="热点参数限流了";
                }else if(e instanceof SystemBlockException){
                    message="触发系统保护规则了";
                }else if(e instanceof AuthorityException){
                    message="服务授权不通过";
                }
                map.put("message", message);
                return ServerResponse.status(HttpStatus.OK)
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(BodyInserters.fromValue(map));
            }
        };
        GatewayCallbackManager.setBlockHandler(blockRequestHandler);
    }
}
~~~
---
 