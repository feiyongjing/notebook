
默认情况下，如果Eureka Server在一定时间内（默认90秒）没有接收到某个微服务实例的心跳，Eureka Server将会移除该实例。
但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，而微服务本身是正常运行的，此时不应该移除这个微服务，
所以引入了自我保护机制来保留微服务，即15分钟内心跳失败低于85%的服务会进入自我保护不会踢出

# 使用Eureka注册中心

### 1. 新建一个Spring Boot项目

### 2. 添加Eureka 服务端依赖，以及在项目启动类上添加@EnableEurekaServer注解
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~
~~~java
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
~~~

### 3. 添加相关的配置信息
~~~yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost # 注册中心的主机名
  client:
    register-with-eureka: false # false让注册中心不要注册自己
    fetch-registry: false # 表示不去服务端检索其他服务信息，注册中心的职责是维护服务实例
    service-url:
      defaultZone: http://eureka8763:8763/eureka,http://eureka8764:8764/eureka # 指定注册中心的地址
  server:
#    enable-self-preservation: false 禁止使用自我保护机制，默认是开启的
~~~

# 微服务连接Eureka注册中心

### 1. 服务端添加Eureka 客户端的依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>2.2.10.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>
~~~

### 2. 服务端项目添加yml或properties配置
~~~yaml
eureka:
  instance:
    lease-renewal-interval-in-seconds: 2 # 客户端每2s发送一次心跳
    lease-expiration-duration-in-seconds: 10 # 10s 没有发送心跳说明客户端故障，服务会被踢出去
    prefer-ip-address: true # true说明客户端服务的服务实例以IP作为链接，而不是域名
    instance-id: spring-cloud-goods-8081 # 告诉注册中心服务实例的id，id是唯一的
  client:
    service-url:
      defaultZone: 'http://admin:123456@eureka8762:8762/eureka,http://admin:123456@eureka8763:8763/eureka,http://admin:123456@eureka8764:8764/eureka' # 注册中心服务的连接地址
~~~

### 3. 服务端项目启动类添加@EnableEurekaClient注解 ，高版本可以不添加

 
# Eureka的安全认证
给Eureka注册中心添加安全认证，不让外来的服务随便的注册

### 1. 给注册中心项目添加依赖
~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.3.12.RELEASE</version>
</dependency>
~~~

### 2. 给注册中心项目设置用户名和密码
~~~yml
spring:
  security:
    user:
      name: admin             # 注册中心设置用户名
      password: 123456        # 注册中心设置密码
~~~

### 3. （Spring Boot 2.0以下可以不用进行这一步）创建一个Config的配置类，添加@EnableWebSecurity注解和继承WebSecurityConfigurerAdapter类重写configure方法，设置csrf防御通过机制
~~~java
@Configuration
@EnableWebSecurity
public class EurekaSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception{
        // 方式一
        // http.csrf().disable();  直接将csrf置为不可用，并且将Spring 的安全认证也关闭了，即所有请求都可以访问
        
        // 方式二 将csrf置为不可用，但是Spring 的安全认证仍然开启
        // http.csrf().disable()
        // .authorizeRequests()
        // .anyRequest()
        // .authenticated()
        // .and()
        // .httpBasic();
        
        // 方式三
        http.csrf().ignoringAntMatchers("/eureka/**"); // 设置csrf指定路径不检查token直接放行，但是Spring 的安全认证仍然开启
        super.configure(http);
    }
}
~~~
第四步：将所有项目的所有配置文件中连接该注册中心的url都进行替换，就算是注册中心之间相互注册的url也要替换
例子如下：
原来的注册中心连接url
~~~yml
eureka:
  client:
    service-url:
      defaultZone: 'http://eureka8762:8762/eureka,http://eureka8763:8763/eureka,http://eureka8764:8764/eureka'
~~~
---

替换后的注册中心连接url，即在域名前添加   用户名:密码@
~~~yml
eureka:
  client:
    service-url:
      defaultZone: 'http://admin:123456@eureka8762:8762/eureka,http://admin:123456@eureka8763:8763/eureka,http://admin:123456@eureka8764:8764/eureka'
~~~
 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 