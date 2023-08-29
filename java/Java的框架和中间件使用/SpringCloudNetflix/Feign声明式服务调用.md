### Feign是Netflix开发的声明式、模板化的HTTP客户端， Feign可以帮助我们更快捷、优雅地调用HTTP API。
在Spring Cloud中，使用Feign非常简单——创建一个接口，并在接口上添加一些注解，代码就完成了。
Feign支持多种注解，例如Feign自带的注解或者JAX-RS注解等。
### Feign=ribbon + restTemplate
Spring Cloud对Feign进行了增强，使Feign支持了Spring MVC注解，并整合了Ribbon和Eureka，从而让Feign的使用更加方便。

# 使用Feign

### 1. 若没有添加Feign的依赖，则需要添加Feign的依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

### 2. 创建一个Feign接口（这个接口一般是在通用的父项目中）
~~~java
@FeignClient(name="SPRING-CLOUD-GOODS", // 声明调用微服务的名字
             // path="/goods", 是服务提供者的Controller类@RequestMapping("/goods")的路径，可以不使用
)
public interface GoodsRemoteClient {
    /**
     * 声明一个Feign的接口，它的实现是服务提供者的Controller层实现
     * 微服务的消费者调用当前接口就相当于调用服务提供者的Controller层
     * 注意这里使用的路径映射注解要与服务提供者的注解保证完全一致，
     * 即服务提供者使用@RequestMapping这里就使用@RequestMapping，其他的REST路径映射注解也一样，
     * 否则Feign在整合Hystrix或其他的东西时，服务消费者在调用服务提供者时会出现服务提供者是null出现空指针错误
     */
    @RequestMapping(value = "/goods/test",method = RequestMethod.GET)
    String getTest(@RequestParam("goodsId")String goodsId);
}

~~~

### 3. 项目的启动类上添加@EnableFeignClients

### 4. 使用第二步创建的Feign接口
~~~java
@RestController
@RequestMapping("/parent_goods")
public class TestController {

    // 注入Feign接口
    @Autowired
    GoodsRemoteClient goodsRemoteClient;

    @GetMapping("/parent_test")
    private String getTest1(@RequestParam("goodsId") String goodsId){
        return goodsRemoteClient.getTest(goodsId);  // 使用Feign接口直接调用
    }
} 

~~~
 

# Feign的日志配置

## 分为全局配置和局部配置，全局配置对所有的服务提供者生效，局部配置对部分服务提供者生效

## 全局配置

### 1. 在服务的调用方创建一个Config类，内容如下
~~~java
import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {
    @Bean
    public Logger.Level feignLoggerLevel(){
//        return Logger.Level.NONE; // 不记录日志
//        return Logger.Level.BASIC; // 记录请求方法、URL、响应状态码和执行时间
//        return Logger.Level.HEADERS; // 在BASIC的基础上添加记录请求和响应的header
        return Logger.Level.FULL; // 在HEADERS的基础上添加记录请求和响应的body和元数据
    }
}
~~~

### 2. SpringBoot的默认日志级别是info, Feign的日志级别是debug不会输出，需要配置
~~~yml
logging:
  level:
    FeignInterfacePackage: debug # 注意这个FeignInterfacePackage是指Feign接口所在的包
~~~

---

## 局部配置

### 第一种配置方式

1. 在全局配置的基础上去除全局配置第一步创建的Config类的@Configuration注解

2. 在需要配置日志服务对应的Feign接口上的@FeignClient注解添加configuration属性，示例如下
~~~java
@FeignClient(value = "stock-server",
        path = "/stock",
        configuration = FeignConfig.class
)
~~~

### 第二种配置方式

1. 添加配置
~~~yml
# SpringBoot的默认日志级别是info, Feign的日志级别是debug不会输出
logging:
  level:
    FeignInterfacePackage: debug  # 注意这个FeignInterfacePackage是指Feign接口所在的包

feign:
  client:
    config:
      stock-server:   # stock-server是需要配置日志服务的服务名
        loggerLevel: FULL
~~~
 

### Feign的契约
如果有老的项目使用的Feign的原生注解如@RequestLine(GET /getUser)，但是有不想破坏原有的代码替换为新的注解，则设置如下配置
~~~yml
feign:
  client:
    config:
      stock-server:   # stock-server是需要配置日志服务的服务名
        contract: feign.Contract.Default
~~~
全局的契约配置
~~~java
@Bean
public Contract feignContract(){
    return new Contract.Default();
}
~~~

 