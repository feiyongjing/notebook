### 微服务调用时安全认证的配置
需要修改服务的提供者项目和服务的调用者项目

### 修改服务的提供者项目，有两种方式

# 第一种方式：所有被调用的服务提供者项目添加Spring Security依赖和相关配置

### 1. 被调用的服务提供者项目添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.3.12.RELEASE</version>
</dependency>
~~~
### 2. 被调用的服务提供项目添加配置
~~~yml
spring:
  security:
    user:
      name: cat           # 设置访问该微服务需要的用户名
      password: 123456    # 设置访问该微服务需要的密码
~~~

# 第二种方式：将安全认证抽取成独立的模块，其他的所有被调用的服务提供者项目添加该模块的依赖，以及spring boot添加扫描路径

### 1. 创建一个新的Spring Boot项目模块

### 2. 在第一步创建的项目中创建一个Config类，用于设置安全认证的用户名和密码，例如
~~~java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(AuthenticationManagerBuilder auth)throws Exception{
        auth.inMemoryAuthentication()
                .passwordEncoder(bCryptPasswordEncoder())
                .withUser("cat")
                .password(bCryptPasswordEncoder().encode("123456"))
                .roles("USER")
                .and()
                .withUser("admin")
                .password(bCryptPasswordEncoder().encode("123456"))
                .roles("USER","ADMIN");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.httpBasic().and().authorizeRequests().anyRequest().fullyAuthenticated();
        http.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }

}
~~~

### 3. 对被调用的服务提供项目添加第一步创建的项目依赖

### 4. 若第二步创建的类不在被调用的服务提供项目Spring Boot扫描路径中则需要在启动类上添加扫描路径，例子
~~~java
@SpringBootApplication
@ComponentScan(basePackageClasses = {AuthApplication.class,PortalApplication.class})
public class PortalApplication {

    public static void main(String[] args) {
        SpringApplication.run(PortalApplication.class, args);
    }

}
~~~

## 修改服务的调用者项目，按照服务的调用方式来修改项目

### 若使用RestTemplate方式调用其他服务，则按照如下步骤操作

### 1. 在Config类中添加HTTP头信息配置
~~~java
@Configuration
public class RestConfig {

    /**
     * 进行HTTP头信息配置
     */
    @Bean
    public HttpHeaders getHttpHeaders(){
        // 定义HTTP头部信息
        HttpHeaders httpHeaders=new HttpHeaders();
        // 认证的原始信息，即访问其他服务需要的用户名和密码
        String auth="cat:123456";
        // 进行加密处理
        byte[] encodeAuth = Base64.getEncoder().encode(auth.getBytes(StandardCharsets.US_ASCII));
        String authHeader="Basic "+new String(encodeAuth);
        httpHeaders.set("Authorization",authHeader);
        return httpHeaders;
    }
}
~~~

### 2. 在调用其他服务时需要添加第一步配置的Http头信息
~~~java
@RestController
@RequestMapping("/parent_goods")
public class TestController {
    // 注意SPRING-CLOUD-GOODS是微服务程序的名字，而不是注册中心服务实例的id
    public static String GOODS_SERVICE_URL_2="http://SPRING-CLOUD-GOODS/goods/test?goodsId={goodsId}";

    @Autowired
    RestTemplate restTemplate;
    
    @Autowired
    HttpHeaders httpHeaders;

    @GetMapping("/parent_test")
    private String getTest(@RequestParam("goodsId") String goodsId){
        Map<String,String> params=new HashMap<>();
        params.put("goodsId",goodsId);
        // 第一参数是微服务的url，第二参数是响应类型，第三参数是请求的实际参数
        // ResponseEntity<String> entity = restTemplate.getForEntity(GOODS_SERVICE_URL_2, String.class,params);
        
        // 第一参数是微服务的url，第二参数是请求的方式，第三参数是请求实体内部包含请求头，第四参数是响应类型，第五参数是请求的实际参数
        ResponseEntity<String> entity = restTemplate.exchange(GOODS_SERVICE_URL_2, HttpMethod.GET, new HttpEntity<>(httpHeaders), String.class,params);
        System.out.println(entity.getStatusCode());
        System.out.println(entity.getStatusCodeValue());
        System.out.println(entity.getHeaders());
        System.out.println(entity.getBody());
        return entity.getBody();
    }
}
~~~
---

### 若使用Feign方式调用其他服务，则按照如下步骤操作

### 1. 创建一个新的配置类用于生产BasicAuthRequestInterceptor类型的Bean，例子如下
~~~java
import feign.auth.BasicAuthRequestInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfiguration {
    @Bean
    public BasicAuthRequestInterceptor basicAuthRequestInterceptor(){
        return new BasicAuthRequestInterceptor("cat","123456");  // 传入访问其他服务需要的用户名和密码
    }
}
~~~

### 2. 在声明Feign接口﻿​上的@FeignClient﻿​添加注解configuration属性，值是第一步创建的Class，例子如下
~~~java
@FeignClient(value = "SPRING-CLOUD-GOODS",
        fallbackFactory = GoodsRemoteClientFallBackFactory.class,
        configuration = {FeignConfiguration.class}
)
public interface GoodsRemoteClient {
    /**
     * 声明一个Feign的接口，它的实现是服务提供者的Controller层实现
     * 微服务的消费者调用当前接口就相当于调用服务提供者的Controller层
     */
    @GetMapping("/goods/test")
    String getTest(@RequestParam("goodsId")String goodsId);
}
~~~
