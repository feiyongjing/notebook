### Zuul作为微服务系统的网关组件，是从设备和网站到Netflix流应用程序后端的所有请求的前门。
作为边缘服务应用程序，Zuul旨在实现动态路由，监控，弹性和安全性。
不用zuul，让客户端直接与各个微服务通讯，会有以下的问题：
1. 客户端会多次请求不同的微服务，增加了客户端的复杂性。
2. 存在跨域请求，在一定场景下处理相对复杂。
3. 认证复杂，每个服务都需要独立认证。
4. 难以重构，随着项目的迭代，可能需要重新划分微服务。例如，可能将多个服务合并成一个或者将一个服务拆分成多个。如果客户端直接与微服务通讯，那么重构将会很难实施。
5. 某些微服务可能使用了防火墙/浏览器不友好的协议，直接访问会有一定困难。

### Zuul能干什么?
路由转发、过滤限流、权限验证、记录日志

作用：
1. 易于监控。可在微服务网关收集监控数据并将其推送到外部系统进行分析。使用网关时客户端只与网关交互，降低客户端的调用逻辑的复杂度，同时网关也可以实现认证逻辑简化内部服务的之间相互调用的复杂度。
2. 对不同客户端的支持及数据的聚合，如一个网站有web端，手机端，页面所需的数据有同有异，可以将数据整合或者裁剪，减少客户端的请求次数，比如BFF架构。
3. 可以更好的对项目的微服务封装，可将项目的微服务统一封装在一个内网环境中，减少了客户端与各个微服务之间的交互次数。只通过网关提供服务，同时网关也可以对安全，认证，监控，防御单独强化。
4. 易于认证。可在微服务网关上进行认证。然后再将请求转发到后端的微服务，而无须在每个微服务中进行认证。

# 使用zuul

### 1. 创建一个新的Springboot项目，用于启动zuul网关服务

### 2. 添加zuul依赖和eureka的依赖
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
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

### 3. 添加yml配置
~~~yml
server:
  port: 80 # 服务端口

spring:
  application:
    name: springcloud-zuul # 当前服务的名字

eureka:
  instance:
    lease-renewal-interval-in-seconds: 2 # 每2s发送一次心跳
    lease-expiration-duration-in-seconds: 10 # 超过10s没有心跳就踢出当前服务
    prefer-ip-address: true # true代表以服务ip作为连接，而不是机器名
    instance-id: springcloud-zuul-80 # 当前服务的唯一id
  client:
    service-url:
      defaultZone: http://eureka8762:8762/eureka,http://eureka8763:8763/eureka,http://eureka8764:8764/eureka # 注册中心集群

zuul:
  host:
    connect-timeout-millis: 5000 # zuul连接超时时间
  routes:  # 配置路由规则，zuul网关分发请求给其他的服务
    springcloud-goods-protal: # 这个springcloud-goods-protal是注册中心已注册服务的名字
      servoce-id: springcloud-goods-protal-8082 # 服务的Id
      path: /portal/** # portal替换服务名，**匹配多级目录
  ignored-services: '*' # 禁止客户通过哪些服务名访问服务，*表示禁止所有的服务名
  prefix: /api # 给所有通过zuul网关访问的服务接口添加路径前缀
  ignored-patterns: /**/feign/** # 路由黑名单，匹配该名单的不允许路由访问，例如当前不匹配路径带有feign的url
~~~

### 4. 启动类上添加@EnableZuulProxy注解开启zuul服务

### 5. 启动注册中心集群、被zuul网关分发的项目、zull项目

### 6. 访问zuul项目接口分发到其他的项目，

例如：原来的访问路径：http://localhost:8082/parent_goods/parent_test?goodsId=xxx

            现在的访问路径：http://localhost:80/api/portal/parent_goods/parent_test?goodsId=xxx

现在的访问路径中ip和端口是网关项目的ip和端口，/api/portal是zuul网关设置的前缀和path替换注册中心的服务名，后面的路径是其他项目的接口路径

# 过滤器
Zuul 中的过滤器总共有 4 种类型，且每种类型都有对应的使用场景。

1. pre 可以在请求被路由之前调用。适用于身份认证的场景，认证通过后再继续执行下面的流程。

2. route 在路由请求时被调用。适用于灰度发布场景，在将要路由的时候可以做一些自定义的逻辑。

3. post 在 route 和 error 过滤器之后被调用。这种过滤器将请求路由到达具体的服务之后执行。适用于需要添加响应头，记录响应日志等应用场景。

4. error 处理请求时发生错误时被调用。在执行过程中发送错误时会进入 error 过滤器，可以用来统一记录错误信息。

 

整个执行的顺序，请求发过来首先到 pre 过滤器，再到 routing 过滤器，最后到 post 过滤器，任何一个过滤器有异常都会进入 error 过滤器。

# 使用过滤器

### 创建过滤器类就行了
~~~java
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import com.netflix.zuul.exception.ZuulException;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;

@Component // 注意这是一个组件
public class LogFilter extends ZuulFilter {
    /**
     * 过滤器类型，可选值有 pre、route、post、error。根据不同的业务使用不同的类型
     */
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    /**
     * 过滤器的执行顺序，数值越小，优先级越高。根据多个业务的优先级来决定
     */
    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER;
    }

    /**
     * 是否执行该过滤器，true 为执行，false 为不执行，这个也可以利用配置中心来实现，达到动态的开启和关闭过滤器。
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 执行自己的业务过滤逻辑
     * @return 返回值是没有意义的，返回null就可以了
     */
    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request=ctx.getRequest();
        System.out.println("访问的地址："+request.getServerName()+request.getRequestURI());
        return null;
    }
}
~~~

## 禁止使用某些过滤器
添加配置
~~~yml
zuul:
  LogFilter:  # 指定禁用过滤器的类名，即禁用LogFilter这个过滤器
    pre: # 指定禁用过滤器的类型，即LogFilter类中filterType方法指定的类型
      disable: true # true禁用过滤器
~~~

如果需要使用自定义的错误处理过滤器需要先禁止SendErroeFilter默认错误处理过滤器，然后自定义error类型的过滤器就行了
~~~yml
zuul:
  SendErrorFilter:
    error:
      disable: true
~~~
