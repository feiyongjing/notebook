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
- 私人仓库
 - 在引用依赖时会先从私人仓库进行读取，如果未找到再从中央仓库下载至私人仓库，最后再下载到本地仓库
- Maven的包
 - 按照约定为所有的包编号，⽅便检索
 - groupId/artifactId/version
   - 扩展：语义化版本
 - 依赖的scope属性
   - 默认是compile，代表编译时需要用到该jar包
   - runtime，代表编译时不需要用到该jar包，但是运行时需要用到该jar包
   - provided，编译时需要用到，但是运行时由JDK或者某个服务器提供
   - test，编译Test时需要用到该jar包
 - 通过 properties 标签即可自定义变量配置，然后使用 ${} 引用变量
 - 项目包含多个子项目时，通过 modules 标签即可实现子模块管理
 - 通过 parent 即可标记当前模块的父模块，且子模块将会继承父模块中的所有依赖配置。子模块若没有指定的 groupId 和 version 默认继承父模块中的配置。
其中 relativePath 用于指定父模块的 POM 文件目录，省略时默认值为 ../pom.xml 即当前目录的上一级中，若仍未找到则会在本地仓库中寻找
 - 当一共项目包含多个模块，且多个模块引用了相同依赖时显然重复引用是不太合适的，而通过 dependencyManagement 即可很好的解决依赖共用的问题。
将项目依赖统一定义在父模块的 dependencyManagement 标签中，子模块只需继承父模块并在 dependencies 引入所需的依赖，便可自动读取父模块 dependencyManagement 所指定的版本。
dependencyManagement 既不会在当前模块引入依赖，也不会给其子模块引入依赖，但其可以被继承的，只有在子模块下同样声明了该依赖，才会引入到模块中，子模块中只需在依赖中引入 groupId 与 artifactId 即可, 也可以指定版本则会进行覆盖
 - SNAPSHOT快照版本
 - 传递性依赖的⾃动管理
 - 原则：绝对不允许最终的classpath出现同名不同版本的jar包
 - 依赖冲突的解决：原则：最近的胜出，如果一样近则先使用的胜出。查看项目所有的依赖（Maven默认的解决原则后的）命令 mvn dependency:true 
    - 解决方案1：找到冲突的所有jar包，将需要的jar包移到与项目路径中最短的地方即直接在pom.xml引入需要的jar包
    - 解决方案2：找到冲突的所有jar包，将不需要的jar包在pom.xml中使用exclusions标签排查掉
 - 当你看到如下的异常的时候
   - AbstractMethodError
   - NoClassDefFoundError
   - ClassNotFoundException
   - LinkageError
 



