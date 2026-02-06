# Gradle项目结构
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
	id 'io.spring.dependency-management' version '1.1.7'
}

// group、name、version 分别对应Maven的 groupId、artifactId、version坐标 
// name一般都是在settings.gradle配置中
group = 'com.example'
version = '0.0.1-SNAPSHOT'
// 项目描述
description = 'Demo project for Spring Boot'

java {
	toolchain {
		languageVersion = JavaLanguageVersion.of(17)
	}
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

// 声明依赖仓库，这里设置的maven的中央仓库
repositories {
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
	implementation 'org.springframework.boot:spring-boot-starter-webmvc'
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframework.boot:spring-boot-devtools'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
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










