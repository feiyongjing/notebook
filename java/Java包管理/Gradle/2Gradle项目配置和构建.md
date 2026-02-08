## Gradle项目结构
Gradle 项目分为Gradle Groovy项目和Gradle Kotlin项目，它们的不同在于使用不同的DSL进行构建

Groovy和Kotlin的差异
- Groovy 松散灵活、新手友好，Kotlin 严格规范、静态类型更安全
- 小项目和新手选 Groovy，大项目和追求可维护性选 Kotlin

以下是 Gradle 项目的结构，Groovy 和 Kotlin的区别就在于 build 和 settings 这两项核心配置文件的差异
~~~
your-project/
├── gradle/                             // Gradle Wrapper 核心目录
│   └── wrapper/
│       ├── gradle-wrapper.jar          // Wrapper 执行包（无需修改）
│       └── gradle-wrapper.properties   // Gradle 版本配置（核心）
├── src/                                // 源码目录（遵循 Maven 标准）
│   ├── main/                           // 主源码
│   │   ├── java/                       // Java 源码（包结构：com/example/demo）
│   │   ├── resources/                  // 资源文件（配置文件、静态资源）
│   │   │   ├── application.yml         // 应用配置（如数据库、端口）
│   │   │   └── logback.xml             // 日志配置
│   │   └── kotlin/                     // 可选：Kotlin 源码（若用 Kotlin 开发）
│   └── test/                           // 测试源码
│       ├── java/                       // 单元测试/集成测试源码
│       ├── resources/                  // 测试专用资源（如测试用配置）
│       │   └── application-test.yml
│       └── kotlin/                     // 可选：Kotlin 测试源码
├── build.gradle                        // Groovy DSL 核心构建脚本（Groovy DSL和Kotlin DSL二选一）
├── build.gradle.kts                    // Kotlin DSL 核心构建脚本（Groovy DSL和Kotlin DSL二选一）
├── settings.gradle                     // Groovy DSL 版本的项目配置（Groovy DSL和Kotlin DSL二选一）
├── settings.gradle.kts                 // Kotlin DSL 版本的项目配置（Groovy DSL和Kotlin DSL二选一）
├── gradlew                             // Linux/Mac 下 Gradle wrapper执行脚本（无需修改）
├── gradlew.bat                         // Windows 下 Gradle wrapper执行脚本（无需修改）
└── .gitignore                          // Git 忽略文件（排除 build、.gradle 等）
~~~

## Gradle Wrapper 的使用
Gradle Wrapper 与 Maven Wrapper的作用类似，确保每个人在项目中都能使用相同的 Gradle 版本
1. 只需要配置 gradle/wrapper/gradle-wrapper.properties 就可以使用 gradlew 进行构建项目了
~~~shell
# 配置Gradle Wrapper使用的gradle的下载链接
distributionUrl=https\://services.gradle.org/distributions/gradle-7.6.6-all.zip

# 配置Gradle Wrapper下载的gradle和后续该gradle下载的依赖放在哪里，只需要配置好GRADLE_USER_HOME即可，这个环境变量一般在本地安装gradle时已经配置好了，所以下面这些配置都不用动
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStorePath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
~~~

2. 使用mvnw替换mvn命令就可以使用 Gradle Wrapper 指定的gradle版本构建项目了
~~~shell
./gradlew clean build
~~~


## settings.gradle和build.gradle配置文件

### settings.gradle配置文件
~~~groovy
// 基本只有项目名称，rootProject.name表示的根项目名称
rootProject.name = 'demo'
~~~

### build.gradle配置文件
gradle中相关名词
1. group、name、version：group是module所在组;name是module的名字，同个组里name具有唯一性;version是module的版本号;group、name和version三者构成了该module的唯一性
2. apply：在module中应用一个插件
3. dependencies：用来声明module所依赖的jar包或其他module
4. repositories：声明一个仓库，告诉程序到哪个仓库去找相应的module、jar包等依赖
5. task:用来声明module的任务，其对应org.gradle.api.Task

~~~groovy
// 定义有那些插件
plugins {
	id 'java'
	id 'org.springframework.boot' version '2.7.8'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'
}

// group、name、version 分别对应Maven的 groupId、artifactId、version坐标 
// name一般都是在settings.gradle配置中
group = 'com.example'
version = '0.0.1-SNAPSHOT'
// 项目描述
description = 'Demo project for Spring Boot'

// Gradle 7.6.6 推荐的 JDK 8 配置方式
sourceCompatibility = 1.8
targetCompatibility = 1.8

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

// 声明依赖仓库，这里设置的阿里云仓库和maven的中央仓库
repositories {
    maven { url 'https://maven.aliyun.com/repository/public' }
	mavenCentral()
}

// 声明需要那些依赖
// gradle依赖的粒度控制相较于Maven也更加精细，maven只有compile、provided、test、runtime四种scope，而gradle有以下几种scope
// 1.implementation，默认的scope。implementation的作用域会让依赖在编译和运行时均包含在内，但是不会暴露在类库使用者的编译时。举例，如果我们的类库包含了gson，那么其他人使用我们的类库时，编译时不会出现gson的依赖。
// 2.api，和implementation类似，都是编译和运行时都可见的依赖。但是api允许我们将自己类库的依赖暴露给我们类库的使用者。
// 3.compileOnly和runtimeonly，这两种顾名思义，一种只在编译时可见，一种只在运行时可见。而runtimeOnly和Maven的provided比较接近,
// 4.testlmplementation，这种依赖在测试编译时和运行时可见，类似于Maven的test作用域
// 5.testCompileOnly和testRuntimeOnly，这两种类似于compileOnly和runtimeOnly，但是作用于测试编译时和运行时
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

// 定义task任务，这里定义的测试任务
tasks.named('test') {
	useJUnitPlatform()
}
~~~


## Groovy 语法

### 配置 Groovy 运行环境方便学习
1. 到 https://groovy.apache.org/download.html 下载对应二进制版本解压缩，我这里Gradle7.6.6使用的Groovy 3.0.17，直接选择Groovy 3.0中的版本就好了
2. 配置java环境变量(如果已经有java环境可以跳过这一步配置)：新增JAVA_HOME环境变量，值为 jdk 解压路径，同时编辑 Path 环境变量，添加 %JAVA_HOME%\bin
3. 配置Groovy环境变量：新增 GROOVY_HOME 环境变量，值为 Groovy 解压路径，同时编辑 Path 环境变量，添加 %GROOVY_HOME%\bin
4. 验证安装：新开cmd窗口执行 groovy -v 命令可以查看安装的Groovy版本

### Groovy的基本语法
~~~groovy
class HelloWorld {

    // 默认不写方法的修饰符就是public
    // 由于groovy是脚本语言，可以不用在main方法中写代码运行，直接的任意一段脚本都可以运行
    static void main(String[] args) {
        // 所有语句末尾都可以省略不写分号
        println("groovy hello world!")

        // 断言，如果是false直接抛异常
        //assert 1+2 == 6

        // 变量可以直接def定义，类型可以省略不写，基本数据类型和java是一致的
        def a = 1
        def int b = 2
        def c = "www"

        // 在Groovy中有两种字符串类型，普通字符串sting(iava.lang.sting)和插值字符串Gsting(groowy.lang.Gsting)
        // 在Groovy中单引号字符串、双引号字符串、三引号字符串都可以定义一个字符串常量，不过只有双引号字符串支持插值。占位符表达式为 ${}或者以 $为前缀
        // 三引号字符串可以保留文本的换行和缩进格式，不支持插值。
        def name ='Android进阶之光'
        def name1 = '''臣本布衣，躬耕于南阳，
                        苟全性命于乱世，
                        不求闻达于诸侯'''
        println name
        println name1
        println "hello ${name}"
        println "hello $name"

        // Groovy没有定义自己的集合类，它在java集合类的基础上进行了增强和简化。
        // Groovy的List对应Java中的List接口，默认的实现类为Java中的Arraylist，可以使用 as 显示声明创建List的实现类
        // 直接使用索引（负索引也没有问题）可以获取列表内部元素
        def number=[1, 2, 3, 4]
        assert number instanceof List
        assert number[1] == 2
        assert number[-1] == 4

        // 添加元素：在列表末尾添加一个元素
        number << 5
        assert number[4] == 5

        def linkedList =[1,2,3] as LinkedList
        assert linkedList instanceof java.util.LinkedList


        // 创建Map同样使用[]，需要同时指定键和值，默认的实现类为java.util.LinkedHashMap。
        def name2 = [one:'魏无羡',two:'杨影枫',three:"张无忌"]
        assert name2['one'] == '魏无羡'
        assert name2.two ==  '杨影枫'
        // Map还有一个键关联的问题
        // 注释1处魏无羡的键值是key这个字符串，而不是key变量的值 name。
        // 如果想要以key变量的值为键值，需要像注释2处一样使用(key)，用来告诉解析器我们传递的是一个变量，而不是定义一个字符串键值。
        def key ='name'
        def person=[key:'魏无羡']//1
        assert person.containsKey('key')
        person=[(key):'张无忌']//2
        assert person.containsKey('name')

        // 方法的调用可以省略括号
        add(a, b)
        minus a, b

        // 实例化对象并且调用自定义的方法和默认提供的成员属性的get就set方法
        User user = new User()
        user.setName "张三"
        user.increaseAge 12
        println "name=" + user.getName() + ", age=" + user.getAge()


        // 多种for循环
        //遍历范围
        def x = 0
        for (i in 0..3) {
            x += i
        }
        assert x == 6

        //遍历列表
        x = 0
        for (i in [0, 1, 2, 3, 4]) {
            x += i
        }
        assert x == 10

        //遍历Map中的值
        x = 0
        def map = ['a': 2, 'b': 3, 'c': 4]
        for (v in map.values()) {
            x += v
        }
        assert x == 9


        // switch多分支判断
        x= 16
        def result =""
        switch(x) {
            case "ok":
                result = "found ok"
                break
            case [1, 2, 4, 'list']:
                result = "list"
                break
            case 10..19:
                result = "range"
                break
            case Integer:
                result = "integer"
                break
            default:
                result = "default"
        }
        assert result == "range"

    }

    // 方法也是使用def定义，当然static还是表示的静态
    static def int add(int a, int b) {
        return a + b
    }

    // 方法如果声明返回值类型可以省略 def，方法参数的类型可以省略，如果省略return那方法的返回值就是方法最后一行的值
    static int minus(a, b) {
        a - b
    }

    // class的定义和jave的基本一样，不过默认修饰符是public，类名也可以与源文件相同，但是还是建议一致，并且会自动提供类成员属性的get就set方法
    static class User {
        String name
        Integer age = 10

        def increaseAge(Integer age) {
            this.age += age
        }
    }

}
~~~

### Groovy的闭包
Groovy中的闭包是一个开放的、匿名的、可以接受参数和返回值的代码块，和Lambda 表达式很类似。
闭包的本质是groovy.lang.Cloush类的一个实例，这使得闭包可以赋值给变量或字段。
~~~groovy
// 闭包定义遵循的语法如下
{[closureParameters -> ] statements }

// 闭包分为两个部分，分别是参数列表部分[closureParameters -> ]和语句部分 statements
// 参数列表部分是可选的，如果闭包只有一个参数，参数名是可选的，Groovy会隐式指定it作为参数名（注意只有it这个参数名才行），如下所示。
{println it}
// 当然这是一个匿名的闭包，以下是闭包的完整写法和调用，除开it的隐式参数外如果使用其他的参数名或者多个参数必须使用类似 Lambda 表达式的写法
// 调用闭包和调用方法基本是一样的
def a = {name ->println name}
assert a instanceof Closure
a "张三"

// 多参数的案例
def b = {String m, String n -> println m+n}
b "你","吃饭没？"

// 闭包既可以当做方法来调用，也可以显示调用call方法
def code ={ 123 }
assert code() == 123 //闭包当做方法调用
assert code.call() == 123 //显示调用ca11方法
def isOddNumber={int i -> i%2 != 0 }
assert isOddNumber(3)==true //调用带参数的闭包


// 设置闭包函数，接受闭包参数并在内部调用闭包
def test(Closure closure) {
    int k = 100
    int v = 200
    println "test1 start..."
    def result = closure(k, v) //调用闭包
    println("result:" + result)
    println "test1 end..."
}
// 调用闭包函数，实际完整是 test({ k, v -> k + v })，只是方法调用省略了外面的括号就变成了下面写法了
test { k, v -> k + v }
~~~

## Gradle项目构建流程
Gradle 的构建过程是通用的，任何由 Gradle 构建的项目都遵循这个过程。
Gradle 构建分为三个阶段，每个阶段都有自己的职责，每个阶段都完成一部分任务，前一阶段的成果是下一阶段继续执行的前提:
1. Initialization 初始化阶段：按顺序执行 init.gradle ->settings.gradle 脚本，生成 Gradle、Setting、Project 对象
2. Configuration 编译阶段：也叫配置阶段。按顺序执行 root build.gradle ->子项目 build.gradle 脚本，生成 Task 执行流程图
3. Execution 执行阶段：按照 Task 执行图顺序运行每一个 Task，完成一个个步骤，生成最终 APK 文件或者jar 文件等  
整个构建过程，官方在一些节点都设置有hook钩子函数，以便构建阶段添加自定义的逻辑进来，影响构建过程，hook钩子函数可以理解为监听函数，另外也可设置 Linsenr监听函数。

### Gradle Task任务
Task是 Gradle 执行的基本单元
Gnadle中，每一个待编泽的工程都叫一个project，每一个projedt在构建的时候都包含一系列的Task，比如一个Android APK的编译可能包含：java源码编译、资源编泽Task,JIN编泽Task、lint检查Task、打包生成APK的Task、签名TasK等。一个project到底包含多少个Task，其实是由编译脚本指定的插件决定。插件是什么呢?插件就是用来定义Task，并具体执行这些Task的东西
Task我们可以看成一个个任务，这个任务可以是编译java代码、编译C/C++代码、编泽资源文件生成对应的R文件、打包生成jar、aar库文件、签名、混淆、图片压缩、打包生成APK文件、包/资源发布等不同类型的项目 Task 任务种类不同，Gradle 现在可以构建 java、android、web、lib 项目。

Task 是完成一类任务，实际上对应的也是一个对象。而 Task 是由无数个 Action 组成的，Action 代表的是一个个函数、方法，每个 Task 都是一堆 Action 按序组成的执行图，就好像我们在 Class 的 main 函数中按照逻辑调用一系列方法一样。
例如，现在有个需求，要在打包出jar的时候顺便看看jar文件的大小，在gradle中仅需在构建脚本中编写几行代码即可，在其中我们可以以编程方式自定义一些构建任务，因为使用了编程方式，所以这带给了我们极大的灵活性和便捷性;而在Maven中则需要编写Maven插件，复杂程度完全不在一个水平
当然，Maven发展到现在，已经存在了大量的插件，提供了各式各样的功能可人使用。但是在灵活性方面还是无法和Gradle相比。而且Grade也有插件功能，现在发展也十分迅猛，存在了大量非常好用的插件，例如gretty插件。gretty原来是社区插件，后来被官方吸收为官方插件，可以在Tomcat和jetty服务器上运行web项目，比Maven的相关插件功能都强大

虽然gradle可以非常灵活的编写自定义脚本任务，但是其实一般情况下我们不要编写构建脚本，利用现有的插件和任务即可完成相关功能。在IDEA里，也可以轻松的查看当前gradle项目中有多少任务，基本任务如build、test等Maven和Gradle都是相通的。

1. 创建任务
~~~groovy
// 自定义 Task，运行 gradle [task名称] 就会执行task任务
// 也可以 task(hello) {...} 或者 task("hello"){...} 进行定义Task

// 执行 gradle hello 打印顺序如下
// > Configure project :
// hello 配置阶段执行
// > Task :hello
// hello 执行阶段doFirst Action
// hello 执行阶段doLast1 Action
// hello 执行阶段doLast2 Action
// hello 执行阶段doLast3 Action

task hello {
	
	// 配置阶段是读取配置时就会执行，也就是运行任何gradle命令时都会读取配置执行
    println "hello 配置阶段执行"

    // doFirst Action执行阶段
    doFirst {
		println "hello 执行阶段 doFirst Action"
	}

    // doLast Action执行阶段
	doLast {
		println "hello 执行阶段doLast Action"
	}
}

// 同理可以设置doFirst Action
hello.configure{
	doLast {
		println "hello 执行阶段doLast2 Action"
	}
}

// 同理可以设置doFirst Action
hello.doLast {
		println "hello 执行阶段doLast3 Action"
}
~~~

2. 任务间共享变量
~~~groovy
// 执行 gradle s1 s2 先执行s1 修改共享变量，然后执行s2打印共享变量
ext { name = "AAA" }
def age = 18
task s1 {
    doLast {
        println("age -->" + age)
        println("name -->" + rootProject.ext.name)
        age = 12
        rootProject.ext.name = "BBB"
        println("This is s1...")
    }
}

task s2 {
    doLast {
        println("age -->" + age)
        println("name -->" + rootProject.ext.name)
        println("This is s2...")
    }
}
~~~

3. 任务依赖：保证任务的执行顺序
Task 依赖是指我们可以指定 Task 之间的执行顺序，重点理解 dependsOn 就行了
- A.dependsOn B-->执行A的时候先执行B
- A.mustRunAfter B->同时执行 A和B时，先执行B再执行A，若执行关系不成立报错
- A.shouldRunAfter B-->同 mustRunAfter，但是执行关系不成立不会报错
~~~groovy
task s1 {
    doLast {
        println("This is s1...")
    }
}

task s2 {
    doLast {
        println("This is s2...")
    }
}

// 执行 gradle s1 时会先执行 s2 任务
s1.dependsOn s2
~~~

4. 自定义Task的Action
~~~groovy
// 实际上创建的Task都是DefaultTask，所以可以继承DefaultTask自定义Task，然后设置Action
class MyTask extends DefaultTask {
    @TaskAction
    def ss1() {
        println("This is MyTask --> action1!")
    }

    @TaskAction
    def ss2() {
        println("This is MyTask --> action2!")
    }
}

// 执行gradle speak 打印如下，会执行自定义的Action
// > Configure project :
// This is AA!
// > Task :speak
// This is doFirst!
// This is MyTask --> action2!
// This is MyTask --> action1!
// This is dolast!

task speak(type: MyTask) {
    println("This is AA!")
    doFirst {
        println("This is doFirst!")
    }
    doLast {
        println("This is dolast!")
    }
}
~~~

5. 默认task会在执行 gradle 无参命令时执行
~~~groovy
// 执行 gradle 会按照默认task列表依次执行task
defaultTasks "s1", "s2"

task s1 {
    doLast {
        println("This is s1...")
    }
}

task s2 {
    doLast {
        println("This is s2...")
    }
}

task s3 {
    doLast {
        println("This is s3...")
    }
}
~~~

5. gradle 本身提供的task或者插件提供task：clean、build等查看方式如下
~~~shell
# 查看所有 Task（含自定义 + 默认），按分类展示
gradle tasks
gradle :应用模块名:tasks

# 查看所有 Task 的详细信息（含描述、依赖）
gradle tasks --all
gradle :应用模块名:tasks --all

# 查看某个 Task 的具体信息（比如查看 build 做了什么）
gradle help --task build
~~~

### Gradle 插件
Gradle 插件本质上是一组已经定义好的Task任务，引用插件就可以使用这些Task

插件都做了什么？
- 将任务添加到项目(如编译、测试)
- 使用有用的默认设置对已添加的任务进行预配置
- 向项目中添加依赖配置
- 通过扩展对现有类型添加新的属性和方法

1. 插件的引入
~~~groovy
// 插件的引入方式有多种

// 方式一
plugins {
    id 'java'
}

// 方式二
apply plugin: 'java'

// 方式三
apply plugin: org.gradle.api.plugins.JavaPlugin

// 方式四
apply plugin: JavaPlugin
~~~

2. java插件文档参考：https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin
~~~shell
# java插件常见task如下
clean	                删除项目 build 目录（清空所有构建产物）	
build	                完整构建流程：编译 → 测试 → 打包
assemble	            仅打包（不执行测试）
check	                执行所有检查类 Task（核心是单元测试）

jar	                    打包主代码 + 资源为普通 JAR 包，无主类（不可直接 java -jar 运行），仅包含编译产物，jar包在 build/libs/项目名-版本.jar
sourcesJar（需手动启用） 打包源码为 JAR 包，需配置：java { withSourcesJar() }，jar包在 build/libs/项目名-版本-sources.jar
javadocJar（需手动启用） 打包 Javadoc 文档为 JAR 包，jar包在 build/libs/项目名-版本-javadoc.jar

compileJava	            编译 src/main/java 下的主Java源码，class文件在build/classes/java/main
compileTestJava	        编译 src/test/java 下的测试Java源码，class文件在build/classes/java/test
classes	                聚合 Task：compileJava + processResources
testClasses	            聚合 Task：compileTestJava + processTestResources

processResources	    处理主资源文件（复制 + 过滤变量），处理后的资源中 build/resources/main
processTestResources	处理测试资源文件 处理后的资源中 build/resources/test

test	                执行 src/test/java 下的单元测试（默认支持 JUnit 4/5）

javadoc	                生成主代码的 Javadoc 文档在 build/docs/javadoc
testJavadoc	            生成测试代码的 Javadoc 文档在 build/docs/testJavadoc

#  Task 之间有固定的依赖链，执行高等级 Task 会自动触发低等级 Task
build = assemble + check
assemble = jar + classes + compileJava + processResources
check = test + testClasses + compileTestJava + processTestResources

# 举个例子：执行 gradlew test 会自动依次执行：
compileJava → processResources → compileTestJava → processTestResources → test
~~~

3. Spring Boot 插件常用task
~~~shell
# Spring Boot 常用插件task如下
bootJar	    打包为可执行 JAR 包（包含依赖、主类，可直接 java -jar 运行）
bootRun	    直接启动 Spring Boot 应用（无需先打包，开发时常用）
bootClean	清理 Spring Boot 构建产物（兼容 clean Task）

# 多模块时启动其中一个模块
gradle :应用模块名:bootRun
~~~







