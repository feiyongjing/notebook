### 读取配置文件的属性：https://blog.csdn.net/Alian_1223/article/details/118891954 

# 常见注解

### 配置
~~~java
// 配置类
@Configuration    

// 配置Bean
@Bean                   

// 配置Bean的作用域
@Scope                 

// 放在组件类上，会延迟组件Bean的加载，即第一次使用Bean时加载，放在@Configuration类上时其中所有的@Bean方法会延迟加载
@Lazy                    

// bean初始化之后会调用的方法上
@PostConstruct          

// 销毁bean之前进行的方法上
@PreDestroy               
~~~
---

### 自动注入
~~~java
// 自动注入属性
@Autowired       

// @Autowired注解判断多个bean类型相同时，就需要添加使用 @Qualifier("xxBean") 指定特定的Bean的名字
@Qualifier          

// @Resource(name="xxBean")   根据Bean的名字注入，@Resources不使用属性指定Bean的名字就按照首字母小写的类名注入
@Resources       

// @Inject就相当于@Autowired
@Inject               

// @Named就相当于 @Qualifier, 
@Named           

// 自动注入配置属性
@Value("${wxshop.redis.host}")     
~~~
---
 

### 业务逻辑层(组件)
~~~java
// 组件
@Conponent 

// 控制层
@Controller   

// 服务层
@Service        

// 数据层
@Repository  
~~~
---
 

### 控制层的全局配置
~~~java
// 标注的类是控制层的全局配置
@ControllerAdvice   

// 标注的方法用于全局处理控制器里的异常
@ExceptionHandler   
~~~
---
 
### 请求路径分发
~~~java
// 映射的路径
@RequestMapping    
~~~
---

### 请求参数处理
~~~java
// 获取x-www-form-urlencoded（form表单)，required属性设置是否参数是否是必要的
@RequestParam  

// 获取URL路径变量
@PathVariable     

// 获取请求的Body解析成参数
@RequestBody   

// 获取请求参数对应的名称的Cookie解析成参数
@CookieValue     

// 此注解用在请求方法的参数上，用于将http请求头部中指定的值绑定到参数上。
@RequestHeader 

// Controller类中方法注解，是分发到当前Controller类所有请求的参数进行类型的转换器，经过转换后才到达对应的请求方法上
@InitBinder          
~~~
---
 

### 返回参数处理
~~~java
@ResponseBody  返回值包装成  JSON （不加注解则默认使用模板视图）
~~~
---
 

### AOP相关
~~~java
// 声明切面类
@Aspect      

// 在切点方法之前执行
@Before       

// 在切点方法之后执行
@After          

// 切点方法返回后执行
@AfterReturning          

// 切点方法抛异常执行
@AfterThrowing           

// 属于环绕增强，能控制切点执行前，执行后，用这个注解后，程序抛异常，会影响@AfterThrowing这个注解
@Around                       
~~~
---
执行顺序：              around > before > around > after > afterReturning

### Spring是如何切换JDK动态代理和CGLIB的 
~~~
spring.aop.proxy-target-class=true(CGLIB)
spring.aop.auto=true(JDK动态代理)
~~~