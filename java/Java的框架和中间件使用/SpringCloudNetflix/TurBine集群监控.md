### TurBine监控集群的Hystrix的使用，然后汇总到仪表盘DashBoard

## 使用TurBine
第一步：创建一个新的Springboot项目，用于启动TurBine服务

第二步：添加依赖
~~~xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.netflix.archaius</groupId>
    <artifactId>archaius-core</artifactId>
    <version>0.7.1</version>
</dependency>
~~~

第二步：添加yml配置
~~~yml
server:
  port: 3722
eureka:
  client:
    register-with-eureka: false #表示当前服务不会注册到注册中心
    service-url:
      defaultZone: http://eureka8762:8762/eureka,http://eureka8763:8763/eureka,http://eureka8764:8764/eureka
turbine:
  app-config: springcloud-goods-protal,springcloud-goods-protal-1 # 根据注册中心的名字对使用hystrix的项目集群聚合汇总
  cluster-name-expression: new String("default")​
~~~

第三步：启动类上添加@EnableTurbine注解开启服务

第四步：启动TurBine项目、DashBoard项目、被DashBoard监控的项目集群

第五步：先访问被DashBoard监控项目中使用了Hystrix的接口，然后访问DashBoard项目的/hystrix接口，例如：http://localhost:3721/hystrix

再输入TurBine项目的监控接口/turbine.stream，例如：http://localhost:3722/turbine.stream   就可以查看到hystrix的使用信息了

