### 什么是包？
- 包就是把许多类放在⼀起打的压缩包
### JVM的⼯作被设计地相当简单：
- 执⾏⼀个类的字节码
- 假如这个过程中碰到了新的类，加载它
- 那么，去哪⾥加载这些类呢？
 - 类路径（Classpath）
   - -classpath/-cp
   - 类的全限定类名（⽬录层级）唯⼀确定了⼀个类      
### 传递性依赖
- 你依赖的类还依赖了别的类
- Classpath hell
 - 全限定类名是类的唯⼀标识
 - 当多个同名类同时出现在Classpath中，就是噩梦的开始
### 什么是包管理？
- 你要使⽤⼀些第三⽅类，总要告诉JVM从哪⾥找吧？
 - 包管理的本质就是告诉JVM如何找到所需的第三⽅类库
 - 以及成功地解决其中的冲突问题

### 没有maven的启蒙时代
- Apache Ant
 - ⼿动下载jar包，放在⼀个⽬录中
 - 写XML配置，指定编译的源代码⽬录、依赖的jar包、输出⽬录等
- 缺点
 - 每个⼈都要⾃⼰造⼀套轮⼦
 - 依赖的第三⽅类库都需要⼿动下载，费时费⼒
     - 假如你的应⽤依赖了⼀万个第三⽅的类库
 - 没有解决Classpath地狱的问题
### Maven——划时代的包管理
- Convention over configuration
- 约定优于配置
- 必须强调，Maven远远不⽌是包管理⼯具
- Maven的中央仓库
 - 地址 https://repo1.maven.org/maven2/
 - 按照⼀定的约定存储包，需要的时候进行下载
- Maven的本地仓库
 - 默认位于~/.m2
 - 下载的第三⽅包放在这⾥进⾏缓存，不必下次需要的时候在从中央仓库下载
- Maven的包
 - 按照约定为所有的包编号，⽅便检索
 - groupId/artifactId/version
     - 扩展：语义化版本
 - SNAPSHOT快照版本
 - 传递性依赖的⾃动管理
 - 原则：绝对不允许最终的classpath出现同名不同版本的jar包
 - 依赖冲突的解决：原则：最近的胜出，如果一样近则先使用的胜出。查看项目所有的依赖（Maven默认的解决原则后的）命令 mvn dependency:true 
    - 解决方案1：找到冲突的所有jar包，将需要的jar包移到与项目路径中最短的地方即直接在pom.xml引入需要的jar包
    - 解决方案2：找到冲突的所有jar包，将不需要的jar包在pom.xml中使用exclusions标签排查掉
 - 依赖的scope
 - 当你看到如下的异常的时候
   - AbstractMethodError
   - NoClassDefFoundError
   - ClassNotFoundException
   - LinkageError
 



