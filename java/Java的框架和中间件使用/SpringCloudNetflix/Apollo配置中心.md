### Apollo与Spring Cloud Config功能一致，但是它是将配置放入数据库中存储而不是远程的仓库管理

# 使用Apollo配置中心
第一步：启动Apollo的服务端，参考Apollo的代码访问：https://github.com/apolloconfig/apollo-build-scripts ，读README的操作启动Apollo服务端

第二步：访问以启动Apollo服务端的配置中心，默认是8070端口 http://localhost:8070/ 在我的应用中创建一个新的应用

第三步：在第二步中创建的应用在添加微服务的配置，然后发布

第四步：需要读取配置的项目添加Apollo客户端的连接
~~~xml
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.9.1</version>
</dependency>
~~~
第五步：读取配置的项目需要配置Apollo服务端的App Id和Meta Server，有两种方式配置

第一种方式是在bootstrap.yml中添加配置
~~~yml
app: 
  id: spring-cloud-goods # 这是第二步创建的应用app Id
apollo:
  meta: http://192.168.0.100:8080/   # 这是启动Apollo的Config Server的访问路径，在第二步访问页面管理员工具下系统信息查看Config Server的Home Page Url
~~~

第二种方式是在读取配置项目的classpath:/META-INF/app.properties中添加配置
~~~
app.id=spring-cloud-goods
apollo.meta=http://192.168.0.100:8080/
~~~

第六步：在读取配置的项目启动类上添加@EnableApolloConfig


和Spring Cloud Config一样在本地有一份配置文件的缓存，在数据库挂掉后默认使用缓存的配置文件，缓存的路径是C:\opt\data\spring-cloud-goods\config-cache中的文件，spring-cloud-goods是被缓存的项目名称，Linux的路径是windows路径的C盘替换为根目录

相比于Spring Cloud Config的好处是需要修改配置时只需要进入第三步的页面修改配置然后发布就修改成功，不必启动消息队列服务进行配置刷新

