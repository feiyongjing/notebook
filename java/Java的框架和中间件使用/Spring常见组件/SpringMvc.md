# src/main/webapp/WEB-INF/web.xml是web项目的配置文件

web容器的加载顺序ServletContext -> context-param -> listener -> filter -> servlet

1. spring框架解决字符串编码问题：过滤器 CharacterEncodingFilter（filter-name） 
2. 在web.xml配置监听器ContextLoaderListener（listener-class） 
ContextLoaderListener的作用就是启动Web容器时，自动装配ApplicationContext的配置信息。因为它实现了ServletContextListener这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法。 
3. 部署applicationContext的xml文件：contextConfigLocation（context-param下的param-name） 
4. DispatcherServlet是前置控制器，配置在web.xml文件中的。拦截匹配的请求，Servlet拦截匹配规则要自已定义，把拦截下来的请求，依据某某规则分发到目标Controller(我们写的Action)来处理。 
DispatcherServlet（servlet-name、servlet-class、init-param、param-name(contextConfigLocation)、param-value） 
在DispatcherServlet的初始化过程中，框架会在web应用的 WEB-INF文件夹下寻找名为[servlet-name]-servlet.xml 的配置文件，生成文件中定义的bean

~~~xml
<?xml version="1.0" encoding="UTF-8"?>  
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    <context-param><!-- 读取其他的xml配置文件的当前上下文中 -->
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>     
    <filter> <!-- 过滤标签 -->
        <filter-name>characterEncodingFilter</filter-name> <!-- 过滤器的名字 -->
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>  <!-- 过滤器的来源类 -->
        <init-param>  <!-- 初始化过滤器对象的构造器中的一个参数 -->
            <param-name>encoding</param-name>  <!-- 构造器的一个参数的名字 -->
            <param-value>UTF-8</param-value>  <!-- 初始化传入的实际参数 -->
        </init-param>  
        <init-param>  
            <param-name>forceEncoding</param-name>  
            <param-value>true</param-value>  
        </init-param>  
    </filter>  
    <filter-mapping>  <!-- 过滤器的映射，即对不同的过滤器进行映射 -->
        <filter-name>characterEncodingFilter</filter-name>  <!-- 过滤器的名字，即使用那个过滤器 -->
        <url-pattern>/*</url-pattern>  <!-- url路径，对特定的url使用上行的过滤器 -->
    </filter-mapping>  
    <listener>  <!-- ContextLoaderListener的作用就是启动Web容器时，自动装配ApplicationContext的配置信息。因为它实现了ServletContextListener这个接口，在web.xml配置这个监听器，启动容器时，就会默认执行它实现的方法 -->
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>  
    </listener>  
    <context-param>  <!--加载Spring的配置文件，随着监听器触发（必须设置监听器才生效），Spring调用这里，找到Spring的核心配置文件-->
        <param-name>contextConfigLocation</param-name>  
        <param-value>classpath:spring/applicationContext.xml</param-value>  <!-- 指定相应的xml文件名，如果有多个xml文件，可以写在一起并以“,”号分隔。 -->
    </context-param>  
    <servlet-mapping>  <!--写在Servlet标签之前的servlet映射不会进入Spring容器而是Tomcat进行拦截处理 -->
        <servlet-name>default</servlet-name>  <!-- Tomcat的defaultServlet来处理 -->
        <url-pattern>*.css</url-pattern>  <!-- 映射的url -->
    </servlet-mapping>  
    <servlet>  <!-- 配置Servlet，Spring默认的Servlet是DispatcherServlet -->
        <servlet-name>DispatcherServlet</servlet-name><!-- Servlet的名字 -->  
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class> <!-- Servlet的来源类 -->  
        <init-param>  <!-- DispatcherServlet构造器的参数 -->
            <param-name>contextConfigLocation</param-name>  <!-- 参数名字 -->
            <!--其中<param-value>**.xml</param-value> 这里可以使用多种写法-->  
            <!--1、不写,使用默认值:/WEB-INF/<servlet-name>-servlet.xml  / 目录指webapp目录-->  
            <!--2、<param-value>/WEB-INF/classes/dispatcher-servlet.xml</param-value>-->  
            <!--3、<param-value>classpath*:dispatcher-servlet.xml</param-value>-->  
            <!--4、classpath: 是指resources目录 classpath*:是指除了resources目录以外的Jar包中的resources也会被加载  -->
            <!--5、多个值用逗号分隔-->  
            <param-value>classpath:spring/dispatcher-servlet.xml</param-value>  <!-- 指明了配置文件的文件名 -->
        </init-param>  
        <load-on-startup>1</load-on-startup><!--是启动顺序，让这个Servlet随Servletp容器一起启动。-->  
    </servlet>  
    <servlet-mapping>  <!--写在Servlet标签之后的servlet映射进入Spring容器中对应的Servlet处理 -->
        <servlet-name>DispatcherServlet</servlet-name>  <!-- 指定的Servlet来处理 -->
        <url-pattern>/</url-pattern> <!-- 映射的url -->
    </servlet-mapping>  
    <welcome-file-list><!--指定欢迎页面，路径是webapp文件之下 -->  
        <welcome-file>login.html</welcome-file>  
    </welcome-file-list>  
    <error-page> <!--当系统出现404错误，跳转到页面nopage.html，路径也是webapp之下-->  
        <error-code>404</error-code>  
        <location>/WEB-INF/views/nopage.html</location>  
    </error-page>  
    <error-page> <!--当系统出现java.lang.NullPointerException，跳转到页面error.html，路径也是webapp之下-->  
        <exception-type>java.lang.NullPointerException</exception-type>  
        <location>/WEB-INF/views/error.html</location>  
    </error-page>  
    <session-config><!--会话超时配置，单位分钟-->  
        <session-timeout>360</session-timeout>  
    </session-config>  
</web-app>
~~~



# /WEB-INF/<servlet-name>-servlet.xml 负责整个mvc中的配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!--此文件负责整个mvc中的配置-->
    <!-- 自动扫描装配,开启组件扫描,可以指定多个扫描路径，同时不用配置context:annotation-config-->
    <context:component-scan base-package="com.web.service"/>
    <context:component-scan base-package="com.web.controller"/>
    <!--用于激活那些已经在spring容器里注册过的bean，启用spring的一些annotation，比如@Autowired、@Resource 、@PostConstruct、@PreDestroy、@PersistenceContext、@Required -->
    <context:annotation-config/>
    <!-- 配置注解驱动 可以将request参数与绑定到controller参数上 -->
    <mvc:annotation-driven/>
    <mvc:default-servlet-handler/>
    <!--静态资源映射-->
    <mvc:resources mapping="/statics/**" location="/statics/"/>
    <!-- 模型视图视图解析器- -->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/views/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
        <property name="exposeContextBeansAsAttributes" value="true"/>
    </bean>
</beans>
~~~



