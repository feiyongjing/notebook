### SkyWalking 文档中文版：https://skyapm.github.io/document-cn-translation-of-skywalking/ 
 
# Skywalking服务端的搭建
1. 下载Skywalking压缩包: https://skywalking.apache.org/downloads/ 解压缩

2. Skywalking服务分为俩部分服务：OapService收集链路数据和wabappService服务UI界面展示链路数据

在bin目录下运行startup.bat或者是startup.sh启动Skywalking服务会将两部分的服务都运行起来

一般情况下需要wabappService服务的启动端口是8080需要修改，wabappService服务启动端口修改在webapp目录下webapp.yml文件中修改server.port属性
~~~
server.port=8098
~~~
一般情况下需要OapService服务的暴露端口是11800和12800，11800端口是用于收集监控数据、12800端口是用于接收前端请求
OapService服务启动端口修改在config目录下application.yml文件中修改，

接收前端请求端口修改
~~~
core.default.restPort=${SW_CORE_REST_PORT:12800}
~~~
收集监控数据端口修改
~~~
core.default.gRPC=${SW_CORE_GRPC_PORT:11800}​
~~~

3. 访问wabappService服务看到UI界面Skywalking就启动成功，在上述的例子中就是  http://localhost:8098/

 

# Skywalking客户端的连接

### 在需要链路追踪的服务启动时添加JVM参数
-javaagent 指添加agent.jar的绝对路径，agent.jar在下载Skywalking压缩包: https://skywalking.apache.org/downloads/ 解压缩后的agent目录下
-DSW_AGENT_NAME 在Skywalking的UI界面显示的服务名字，一般直接写当前服务的名字
-DSW_AGENT_COLLECTOR_BACKENT_SERVICE 指定Skywalking服务的IP和其中OapService服务收集监控数据的端口

例子如下
~~~
-javaagent:E:\java-project\spring-cloud-alibaba-template\apache-skywalking-apm-es7-8.5.0\apache-skywalking-apm-bin-es7\agent\skywalking-agent.jar
-DSW_AGENT_NAME=gateway-server
-DSW_AGENT_COLLECTOR_BACKENT_SERVICE=127.0.0.1:11800
~~~
 

### 注意如果是对Gateway网关服务进行链路追踪，是显示不出追踪的数据，如果需要显示网关服务的链路追踪数据需要进行如下步骤
1. 在Skywalking服务端打开Skywalking压缩包解压后的agent\optional-plugins目录复制apm-spring-cloud-gateway-2.1.x-plugin-8.5.0.jar或者是其他的gateway的jar包插件到agent\plugins目录下
2. 重启Skywalking服务端和需要链路追踪的Gateway网关服务

 

### Skywalking数据持久化到Mysql
1. 在下载Skywalking压缩包: https://skywalking.apache.org/downloads/ 解压缩后的config目录下application.yml文件中修改下面的属性，其他的不要动
~~~yml
storage:
  selector: ${SW_STORAGE:mysql}
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3310/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:123456}
~~~
2. 查找到对应的数据库驱动jar包放入Skywalking压缩包解压缩后的oap-libs目录下，否则重启Skywalking时其中OapService服务会失败
3. 启动对应的数据库服务，连接数据库创建对应的swtest数据库，重启Skywalking就可以了



### Skywalking自定义链路追踪
Skywalking默认只能看见请求调用链路的接口，自定义链路追踪可以让在Skywalking控制台看见请求调用方法、参数、返回值

1. 在需要自定义链路追踪的服务项目中添加依赖
~~~xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>8.5.0</version>  <!-- 版本与Skywalking服务端的版本保持一致 -->
</dependency>
~~~
2. 在需要查看的方法上添加@Trace，则方法可见。注意要看参数返回值的前提是要添加@Trace注解

看返回值添加@Tag(key="xxx", value="returnedObj")  其中key属性的值xxx必须是方法的名字，value属性的值必须是returnedObj

看参数添加@Tag(key="xxx", value="arg[0]") 其中key属性的值xxx必须是方法的名字，value属性的值必须是arg[x]，x代表方法第几个参数

注意当参数和返回值是对象时，参数或者是返回值的类型必须重写toString方法，因为Skywalking显示的就是toString

当有多个参数或者是参数和返回值一起查看时可以使用@Tags，例子如下

~~~java
@Trace
@Tags({@Tag(key = "addOrder",value = "returnedObj"),
            @Tag(key = "addOrder",value = "arg[0]"),
            @Tag(key = "addOrder",value = "arg[1]"),
      })
public String addOrder(@RequestParam("productId") Long productId,
                       @RequestParam("amount") Integer amount) {
    // 业务代码处理
    return "提交订单成功";
}
~~~
 

### Skywalking性能剖析
通过在Skywalking控制台性能剖析创建任务，然后发起请求，对任务进行性能分析，能分析到调用方法栈是哪个方法慢
