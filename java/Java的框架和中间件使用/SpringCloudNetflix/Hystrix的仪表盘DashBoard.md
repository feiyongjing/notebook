### DashBoard仪表盘用于监控项目中Hystrix的使用情况
 
# 如何使用DashBoard仪表盘

第一步：创建一个SpringBoot项目来专门用于DashBoard仪表盘监控Hystrix

第二步：引入DashBoard的依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
~~~

第三步：启动类上添加@EnableHystrixDashBoard注解

第四步：配置yml文件
~~~yml
server:
  port: 3721

hystrix:
  dashboard:
    proxy-stream-allow-list: localhost
~~~

第五步：保证被DashBoard监控的项目含有Hystrix的依赖和actuator的依赖
~~~xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.3.12.RELEASE</version>
</dependency>
~~~

第六步：在被DashBoard监控的项目yml的配置文件中开启spring boot监控端点的访问权限
~~~yml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
~~~

第七步：启动DashBoard项目和被DashBoard监控的项目

第八步：先访问被DashBoard监控项目中使用了Hystrix的接口，然后访问DashBoard项目的/hystrix接口，例如：http://localhost:3721/hystrix

再输入被监控项目的监控接口/actuator/hystrix.stream，例如：http://localhost:8082/actuator/hystrix.stream   就可以查看到hystrix的使用信息了

