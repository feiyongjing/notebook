## maven项目构建常用命令
~~~shell
mvn clean	            # 清理 target 目录
mvn compile	            # 编译源代码
mvn test	            # 运行单元测试
mvn package	            # 打包（生成 JAR/WAR）
mvn install	            # 将当前maven项目安装到本地仓库
mvn deploy	            # 部署到远程仓库（需配置）

mvn clean install       # 清理并构建安装到本地仓库
mvn clean deploy        # 清理并部署

mvn dependency:tree	    # 查看依赖树

# 查看maven依赖树并将输出追加文件末尾
mvn dependency:tree > test.txt

# 查看maven依赖树并将输出覆盖到指定文件
mvn dependency:tree >> test.txt

# 分析项目中的依赖关系，包括未使用的直接和间接依赖
mvn dependency:analyze
# - 项目中直接依赖但是未使用的依赖项（ Unused declared dependencies found）
# - 项目中直接或间接依赖但在编译或测试阶段确实用到的依赖项（ Used undeclared dependencies found）

# 跳过测试的运行和编译直接进行打包
mvn package -Dmaven.test.skip=true 


~~~

## maven项目构建的生命周期
Maven 生命周期定义了项目从清理、构建到发布的完整流程，由一系列有序的 阶段（phase）组成，每个阶段可绑定多个插件目标（plugin goal）来执行具体任务。

三大内置生命周期：
- Clean：清理项目（删除 target/ 等临时文件）。
- Default（Build）：核心构建流程（编译、测试、打包、部署）。
- Site：生成和部署项目文档站点


### Clean 生命周期阶段（常用）
1. pre-clean：清理前准备
2. clean：删除构建输出目录（由maven-clean-plugin:clean插件完成）
3. post-clean：清理后收尾
~~~shell
mvn clean
# 执行某阶段会自动执行之前所有阶段，也就是会执行 pre-site → site → post-site → site-deploy
~~~

### Default 生命周期核心阶段（常用）
1. validate	                验证项目是否正确并且所有必要的信息都可用。
2. initialize	            初始化构建状态，例如设置属性或创建目录。
3. generate-sources	        生成任何源代码以包含在编译中。
4. process-sources	        处理源代码，例如过滤任何值。
5. generate-resources	    生成包含在包中的资源。
6. process-resources	    将资源复制并处理到目标目录中，准备打包。
7. compile	                编译项目的源代码。
8. process-classes	        对编译生成的文件进行后处理，例如对 Java 类进行字节码增强。
9. generate-test-sources	生成任何测试源代码以包含在编译中。
10. process-test-sources	处理测试源代码，例如过滤任何值。
11. generate-test-resources	为测试创建资源。
12. process-test-resources	将资源复制并处理到测试目标目录中。
13. test-compile	        将测试源代码编译到测试目标目录
14. process-test-classes	对测试编译生成的文件进行后处理，例如对 Java 类进行字节码增强。
15. test	                使用合适的单元测试框架运行测试。这些测试不应该要求打包或部署代码。
16. prepare-package	        在实际包装之前执行准备包装所需的任何操作。这通常会导致包的解包、处理版本。
17. package	                获取已编译的代码并将其打包成可分发的格式，例如 JAR。
18. pre-integration-test	在执行集成测试之前执行所需的操作。这可能涉及诸如设置所需环境之类的事情。
19. integration-test	    处理并在必要时将包部署到可以运行集成测试的环境中。
20. post-integration-test	执行集成测试后执行所需的操作。这可能包括清理环境。
21. verify	                运行任何检查以验证包是否有效并符合质量标准。
22. install	                将包安装到本地存储库中，以用作本地其他项目的依赖项。
23. deploy	                在集成或发布环境中完成，将最终包复制到远程存储库以与其他开发人员和项目共享。
~~~shell
mvn package
# 执行某阶段会自动执行之前所有阶段，也就是会执行到package
~~~

### Site 生命周期阶段：
1. pre-site：生成站点前准备
2. site：生成站点文档（由maven-site-plugin:site完成）
3. post-site：生成后处理
4. site-deploy：部署站点到服务器
~~~shell
mvn site-deploy
# 执行某阶段会自动执行之前所有阶段，也就是会执行 pre-site → site → post-site → site-deploy
~~~

## maven插件
默认maven在各个生命周期上绑定有预设的功能，其实本质是maven的插件与生命周期内的阶段绑定，在执行到对应生命周期时执行对应的插件功能，通过插件可以自定义每个生命周期阶段的功能

去 https://maven.apache.org/plugins/index.html 可以看到各种插件，例如各种打包插件，可以打包成jar、war、rar包等

常见的插件
- Maven 打包后的 JAR 文件，解压后 META-INF 目录用于存放工程的元数据信息（例如MANIFEST.MF文件），通过org.apache.maven.plugins 的maven-jar-plugin 插件即可在添加额外信息至打包后的 JAR 文件
- 在普通 Maven 工程打包时默认仅会编译工程中新建的 java 文件并存储其 .class 文件，对于 POM 文件中引用的第三方依赖并不会一同打包，通过 org.apache.maven.plugins 的maven-assembly-plugin 插件即可将 POM 配置中的所有依赖一同打包编译至 JAR 文件中
- Shade 插件的功能更为强大，其提供了两个功能：第一个即与 assembly 类似可实现依赖的打包编译，与 assembly 不同的是 Shade 提供了更灵活的执行策略，可指定需要打包编译的依赖集合。另一个即实现包的重命名功能，我们都知道 Maven 在一共工程中同时引入单个依赖的不同版本但是实际只会使用其中一个版本的依赖，而通过 Shade 插件即可实现二次包装从而绕开该限制
- org.apache.maven.plugins 的 maven-compiler-plugin 插件可指定工程打包编译时使用的 JDK 版本，可根据服务器环境手动修改版本


### spring-boot-maven-plugin 插件使用
spring-boot-maven-plugin 插件把 Spring Boot 项目打包成一个包含所有依赖的可执行 JAR，可以直接用 java -jar 运行
以下是使用案例
1. 引入插件
~~~xml
<!-- 
<dependencies>
        <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope> 设置只在编译时提供，不参与打包运行，就可以不用在下面的插件中进行排除lombok依赖的打包
    </dependency>
</dependencies> 
-->


<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>

                <!-- 多main启动类时必须明确指定主启动类 -->            
                <mainClass>com.example.DemoApplication</mainClass>

                <!-- 最终 JAR 包名称 -->
                <finalName>demo-${project.version}</finalName>

                <!-- 生成可执行 JAR（默认 true） -->
                <executable>true</executable>

                <!-- 排除指定依赖，不打入 JAR -->
                <excludes>
                    <exclude>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
~~~

2. 由于spring-boot-maven-plugin 插件绑定在package阶段，所以执行mvn package就可以触发插件完成打包，会在target目录下出现打包好的jar包
~~~shell
# 清理并跳过测试阶段完成打包
mvn clean package -Dmaven.test.skip=true
~~~








