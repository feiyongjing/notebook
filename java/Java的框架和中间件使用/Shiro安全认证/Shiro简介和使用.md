在概念层，Shiro 架构包含三个主要的理念：Subject,     SecurityManager   和   Realm

如图：https://pic2.zhimg.com/80/v2-c0841dfc8cb19a94c322eef635371cf6_720w.jpg

- Subject（主体）：代表单个应用程序用户的状态和安全操作。 这些操作包括身份验证（登录/注销），授权（访问控制）和会话访问。它是Shiro的单用户安全功能的主要机制。
- SecurityManager：管理所有Subject，在单个应用程序中为所有用户执行所有安全性操作。
- Realms：用于进行权限信息的验证，我们自己实现。Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。
  - Realms的Authentication 和 Authorization。其中 Authentication 是用来验证用户身份，Authorization 是授权访问控制，用于对用户进行的操作授权，证明该用户是否允许进行当前操作，如访问某个链接，某个资源文件等。

Subject(主体)的获取：
~~~java
Subject currentUser = SecurityUtils.getSubject();
~~~

参考资料： https://zhuanlan.zhihu.com/p/54176956

https://blog.csdn.net/qq_45275875/article/details/105528547 