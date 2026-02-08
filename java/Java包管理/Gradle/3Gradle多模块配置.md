# 多模块项目结构
~~~
demo/
├── gradle/                             // Gradle Wrapper 核心目录
│   └── wrapper/
│       ├── gradle-wrapper.jar          // Wrapper 执行包（无需修改）
│       └── gradle-wrapper.properties   // Gradle 版本配置（核心）
├── module1/                            // 源码目录（遵循 Maven 标准）
│   ├── src/                            // 源码目录（遵循 Maven 标准）
│   │   ├── main/                       // 主源码
│   │   │   ├── java/                   // Java 源码（包结构：com/example/demo）
│   │   │   └── resources/              // 资源文件（配置文件、静态资源）
│   │   └── test/                       // 测试源码
│   │       ├── java/                   // 单元测试/集成测试源码
│   │       └── resources/              // 测试专用资源（如测试用配置）
│   └── build.gradle                    // module1 的build.gradle
├── module2/                            // 源码目录（遵循 Maven 标准）
│   ├── src/                            // 源码目录（遵循 Maven 标准）
│   │   ├── main/                       // 主源码
│   │   │   ├── java/                   // Java 源码（包结构：com/example/demo）
│   │   │   └── resources/              // 资源文件（配置文件、静态资源）
│   │   └── test/                       // 测试源码
│   │       ├── java/                   // 单元测试/集成测试源码
│   │       └── resources/              // 测试专用资源（如测试用配置）
│   └── build.gradle                    // module2 的build.gradle
├── build.gradle                        // root根 build.gradle 配置
├── settings.gradle                     // root根 settings.gradle 配置
├── gradlew                             // Linux/Mac 下 Gradle wrapper执行脚本（无需修改）
└── gradlew.bat                         // Windows 下 Gradle wrapper执行脚本（无需修改）
~~~



### root根 settings.gradle 配置
~~~groovy
// root项目名称和模块名称
rootProject.name = 'demo'
include 'module1'
include 'module2'
~~~

### root根 build.gradle 配置
~~~groovy
buildscript {
    //统一版本管理
    ext {
        //lombok版本
        lombokVersion = '1.18.20'
        //spring boot 版本
        springBootVersion = '2.7.8'
        //mysql版本
        mysqlVersion = '8.0.13'
        //mybatis版本
        mybatisSpringBootVersion = '2.1.1'
    }

    // 设置仓库用于下载插件
    repositories {
        //从前到后顺序执行，找不到就往后找
        mavenLocal()//本地仓库
        maven { url 'https://maven.aliyun.com/repository/public' }//镜像仓库
        mavenCentral()//官方仓库
    }

    dependencies {
        //spring-boot-gradle插件，方便版本管理
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")

    }

}

//全局配置，包括root和其子项目
allprojects {
    apply plugin: 'java'
    group = 'com.example'
    version = '0.0.1-SNAPSHOT'
    sourceCompatibility = 1.8 //java版本
    targetCompatibility = 1.8 //java版本

    tasks.withType(JavaCompile) {
        options.encoding = "UTF-8"
    }
    repositories {
        //从前到后顺序执行，找不到就往后找。
        mavenLocal()//本地仓库
        maven { url 'https://maven.aliyun.com/repository/public' }//镋像仓库
        mavenCentral()//官方仓库
    }
}

// 子模块管理
subprojects {
    apply plugin: 'io.spring.dependency-management' //版本管理插件
    apply plugin: 'org.springframework.boot'

    dependencies {
        annotationProcessor 'org.projectlombok:lombok'//注释处理器
        implementation 'org.projectlombok:lombok'//引入lombok依赖
    }
    //dependencyManagement版本统一管理，类似maven的dependencyManagement
    dependencyManagement {
        dependencies {
            //统一版本管理
            dependency "org.projectlombok:lombok:${lombokVersion}"
            dependency "mysql:mysql-connector-java:${mysqlVersion}"
            dependency "org.mybatis.spring.boot:mybatis-spring-boot-starter:${mybatisSpringBootVersion}"
        }
        // 给所有的子项目导入Spring Boot的dependencyManagement
        imports {
            mavenBom "org.springframework.boot:spring-boot-dependencies:${springBootVersion}"//Spring Boot
        }
    }
}
~~~

### module 的build.gradle配置（module1和module2，多个模块配置基本一样）
~~~groovy
// 只需要写该模块的依赖
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}

// 或者是按照如下写法将模块的依赖写在root根 build.gradle 配置中
// project("module1"){
//     dependencies {
//         implementation 'org.springframework.boot:spring-boot-starter-web'
//     }
// }
~~~


### 运行
~~~shell
# 只有一个模块可以直接运行
gradle bootRun

# 多个模块选择对应模块运行
gradle :应用模块名:bootRun
~~~


