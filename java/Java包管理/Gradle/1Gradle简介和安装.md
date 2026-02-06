# Gradle是什么 
Gradle是一款Google推出的，基于JVM，通用灵活的项目构建工具，支持Maven、JCenter多种第三方仓库；支持传递性依赖管理，废弃了繁杂的xml文件，转而使用简洁的、支持多种语言（Java、Groovy、Kotlin）的build脚本文件

# Gradle安装
1. 在 https://gradle.org/releases/ 选择对应版本的二进制包下载解压缩安装，注意请根据jdk版本选择Gradle版本进行安装，目前Gradle 7.6 是最后一个支持 JDK 8 运行的版本
2. 配置java环境变量(如果已经有java环境可以跳过这一步配置)：新增JAVA_HOME环境变量，值为 jdk 解压路径，同时编辑 Path 环境变量，添加 %JAVA_HOME%\bin
3. 配置Gradle环境变量：新增 GRADLE_HOME 环境变量，值为 Gradle 解压路径，同时编辑 Path 环境变量，添加 %GRADLE_HOME%\bin
4. 新增 GRADLE_USER_HOME 环境变量，值为Gradle本地仓库目录。Gradle 需要下载相关依赖包，默认是在 C 盘里面，后续可能会占用较多磁盘，因此可以增加这个环境变量的配置，修改下载下来的依赖的存储位置和Gradle Wrapper 缓存，可以理解成 Maven 的 localRepository 标签配置。注意Gradle本地仓库可以和Maven本地仓库都是存储相同依赖包的，因此可以共用一个目录存储本地仓库的依赖包
5. 验证安装：新开cmd窗口执行 gradle -v 命令可以查看安装的Gradle版本

### Gradle配置镜像仓库加速配置

在Gradle安装目录中的init.d目录下，创建以.gradle结尾的文件，.gradle文件可以实现在build开始之前执行。所以我们可以在这个文件中配置预先加载的操作。

在init.d目录下，创建一个init.gradle文件
~~~groovy
// 配置参考：https://maven.aliyun.com/mvn/guide
allprojects {
    repositories {
        // 阿里云镜像
        maven { url 'https://maven.aliyun.com/repository/public/' }
        maven { url 'https://maven.aliyun.com/repository/google/' }
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin/' }
        
        // 备用镜像源
        maven { url 'https://mirrors.huaweicloud.com/repository/maven/' }
        // mavenLocal() 就是说明首先使用 maven 的本地仓库获取依赖，如果需要mavenLocal()生效，则需要配置配置M2_HOME环境变量，值是maven的安装路径
        mavenLocal()
        mavenCentral()
    }
    
    buildscript {
        repositories {
            maven { url 'https://maven.aliyun.com/repository/public/' }
            maven { url 'https://maven.aliyun.com/repository/gradle-plugin/' }
        }
    }
}


// mavenLocal() 表示指定使用maven本地仓库，而本地仓库在配置maven时settings文件指定的仓库位置。 
// mavenLocal() 查找jar包顺序如下：USER_HOME/.m2/settings.xml >> M2_HOME/conf/settings.xml >> USER_HOME/.m2/repository
 
// maven { url 地址}，指定maven仓库，一般用私有仓库地址或其它的第三方库，比如阿里镜像仓库地址
// mavenCentral()：这是Maven的中央仓库，无需配置，直接声明就可以使用
 
 
// gradle可以通过指定仓库地址为本地maven仓库地址和远程仓库地址相结合的方式，避免每次都会去远程仓库下载依赖库。这种方式也有一定的问题，如果本地maven仓库有这个依赖，就会从直接加载本地依赖，如果本地仓库没有该依赖，那么还是会从远程下载。但是下载的jar不是存储在本地maven仓库中，而是放在自己的缓存目录中，默认在USER_HOME/.gradle/caches目录,当然如果我们配置过`GRADLE_USER_HOME`环境变量，则会放在`GRADLE_USER_HOME/caches`目录,那么可不可以将gradle caches指向maven repository。这是不行的，caches下载文件不是按照maven仓库中存放的方式

~~~


