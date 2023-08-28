# Spring容器的启动过程

### 主要类
BeanFactory接口的实现类    代表Spring容器本身  方法getBean(String name) 根据名字获取bean
ApplicationContext是BeanFactory接口的子接口    Spring Boot对Spring容器的增强版

### refresh模板方法
读取Bean，从XML或注解读取，并创建Bean的实例
将所有的Bean实例放入Spring容器


# SpringApplication类实例run方法详解，后面附代码

1. 创建一个StopWatch，用于监控启动过程
2. 设置程序为无显示模式
3. 获取监听器SpringApplicationRunListener,用于启动过程的事件广播
4. 广播启动事件(ApplicationStartingEvent)
5. 环境准备 environment 广播
6. 创建上下文
7. 获取SpringBootExceptionReporter , (启动失败的异常信息日志)
8. 准备上下文
9. 刷新上下文
10. 刷新上下文之后的一些扩展 (现在是空的,有需要自己可以重写)
11. 输出启动的时间
12. 广播启动成功事件  
13. callRunners 执行自定义的业务代码(比如业务初始化)  
13. 广播运行成事件  

~~~java
 1public ConfigurableApplicationContext run(String... args) { 
    //创建一个StopWatch，用于监控启动过程 
    StopWatch stopWatch = new StopWatch(); 
    //启动时间戳:纳秒 
    stopWatch.start(); 
    ConfigurableApplicationContext context = null; 
    Collection exceptionReporters = new ArrayList<>(); 
    //设置了系统全局变量 {java.awt.headless, true } 用于在缺失显示屏、鼠标或者键盘时的系统配置,也就是Web服务 
    configureHeadlessProperty(); 
    //获取监听器SpringApplicationRunListener,用于启动时的事件广播 
    SpringApplicationRunListeners listeners = getRunListeners(args); 
    //监听器广播ApplicationStartingEvent事件 即广播应用程序启动事件
    listeners.starting(); 
    try { 
        // 封装应用程序的启动参数 
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args); 
        // 准备环境：命令行参数,系统参数,配置文件,外部配置文件,(非web/web/响应式web)项目环境
        //2、广播ApplicationEnvironmentPreparedEvent事件 即广播应用程序环境准备事件
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments); 
        // 设置系统变量spring.beaninfo.ignore, true 
        configureIgnoreBeanInfo(environment); 
        //打印banner 即打印横幅 默认打印到系统标准输出，方法内部支持打印到日志输出
        Banner printedBanner = printBanner(environment); 
        // 根据不同的应用程序创建不同的ConfigurableApplicationContext 即Spring容器的创建
        context = createApplicationContext(); 
        //创建SpringBootExceptionReporter, 启动异常后的相关处理 
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[] { ConfigurableApplicationContext.class }, context); 
        //准备容器 
        prepareContext(context, environment, listeners, applicationArguments, printedBanner); 
        //刷新上下文 
        refreshContext(context); 
        //刷新上下文之后的一些扩展,现在是空方法 
        afterRefresh(context, applicationArguments); 
        //启动完成计算启动耗时 
        stopWatch.stop(); 
        if (this.logStartupInfo) { 
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        } 
        //广播容器启动成功 
        listeners.started(context); 
        //可以执行一些业务初始化 通过ApplicationRunner,CommandLineRunner callRunners(context, applicationArguments); 
        } catch (Throwable ex) { 
            handleRunFailure(context, ex, exceptionReporters, listeners); throw new IllegalStateException(ex);
            } try { 
                //广播 容器运行 
                listeners.running(context); 
            } catch (Throwable ex) { handleRunFailure(context, ex, exceptionReporters, null); throw new IllegalStateException(ex); 
            } 
            return context; 
} 
~~~
 