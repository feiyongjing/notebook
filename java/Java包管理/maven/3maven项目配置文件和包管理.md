# Maven项目结构
~~~
your-project/
├── pom.xml                                    # maven项目配置文件
├── src/
│   ├── main/
│   │   ├── java/                              # 开发目录
│   │   │   └── Main.java
│   │   └── resources/
│   │       └── application.yml                # 资源文件
│   └── test/                                  # 测试目录
│       ├── java/
│       │   └── Test.java
│       └── resources/
│           └── application.yml                # 资源文件
└── README.md
~~~

## Maven依赖包坐标
Maven依赖包坐标是由groupId、artifactId、version组成的
Maven项目中的所有依赖包都可以Maven坐标通过在Maven仓库中找到
- groupId：依赖包所属的组织名称
- artifactId：依赖包的名称
- version：依赖包的版本
~~~xml
<!--依赖的第三方包声明        -->
<dependency>
    <!--标签放的是组织名称或者公司域名的反写        -->
    <groupId>org.junit.jupiter</groupId>
    <!--标签放的是不同功能的名字        -->
    <artifactId>junit-jupiter-engine</artifactId>
    <!--标签放的是不同的版本号        -->
    <version>5.6.0</version>

    <!-- 默认值是compile，表示当前依赖会参与当前项目的编译、打包、测试、运行等
         test表示当前依赖仅参与测试，在编译、运行、打包时不会使用这个依赖
         runntime表示当前依赖参与打包、运行、测试，在编译时下不会被使用
         provided表示当前参与编译和测试，打包和运行时不需要，例如 spring-boot-devtools 依赖在生产运行是不需要热部署，所有可以设置为这个值
         
         通过设置test和provided不参与打包可以避免一下不是必要的依赖被传递到其他项目，当然如果是必要的依赖也可以通过下面的optional设置依赖不要扩散到其他项目
    -->
    <scope>test</scope>


    <!-- optional用于设置依赖是否应该被传递。
    当A项目依赖于某个jar包，而B项目依赖A项目同时也会传递性的依赖这个jar包。如果不想要把这个jar包的依赖从A项目扩散到其他项目，就在A项目引入该jar包依赖时设置optional为true，则依赖不会被传递到B项目，但是该依赖还是在A项目中
    当然默认不设置optional，默认值就是false表示会传递性的扩散依赖
    -->
    <optional>false</optional>


    <!-- 排除依赖：对于一下完全用不到的依赖或者依赖冲突的情况，使用排除依赖可以减小依赖包的大小和避免依赖冲突 -->
    <exclusions>
        <exclusion>
            <groupId>www</groupId>
            <artifactId>fff</artifactId>
            <version>xxx</version>
        </exclusion>
    </exclusions>
</dependency>
~~~


## maven依赖包管理
传递性依赖的⾃动管理：maven对于依赖包的管理是默认引入一个依赖包会将这个依赖包所依赖的其他依赖包（包括内部递归依赖的其他依赖包也一并引入），但是这样会产生一些问题，例如出现相同的依赖只是版本不同的依赖冲突问题，这会导致classpath出现全限定类名同名class

maven是如果处理出现相同的依赖只是版本不同的问题？
- 原则：绝对不允许最终的classpath出现同名不同版本的jar包，对于相同的依赖只是版本不同，默认是依赖路径最短的胜出，如果一样短则项目中先使用的胜出

当然这并没有完全解决问题，在有些情况下对于依赖路径一样短的依赖其实需要的并不是先使用的依赖版本，对于这种情况有如下两种方法解决
- 解决方案1：找到冲突的所有jar包，将需要的jar包版本移到与项目路径中最短的地方即直接在pom.xml引入需要的jar包
- 解决方案2：找到冲突的所有jar包，将不需要的jar包版本在pom.xml中使用exclusions标签排查掉

## pom.xml 配置文件
~~~xml
<?xml version="1.0" encoding="utf-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

        <!--声明项目描述符遵循哪一个POM模型版本。模型本身的版本很少改变，虽然如此，但它仍然是必不可少的，这是为了当Maven引入了新的特性或者其他模型变更的时候，确保稳定性。-->
        <modelVersion>4.0.0</modelVersion>

        <!--父项目的坐标。如果项目中没有规定某个元素的值，那么父项目中的对应值即为项目的默认值。 坐标包括group ID，artifact ID和 version。-->
        <parent>
            <!--被继承的父项目的构件标识符-->
            <artifactId/>
            <!--被继承的父项目的全球唯一标识符-->
            <groupId/>
            <!--被继承的父项目的版本-->
            <version/>
            <!-- 父项目的pom.xml文件的相对路径。相对路径允许你选择一个不同的路径。默认值是../pom.xml。Maven首先在构建当前项目的地方寻找父项目的pom，其次在文件系统的这个位置（relativePath位置），然后在本地仓库，最后在远程仓库寻找父项目的pom。-->
            <relativePath/>
        </parent>


        <!--项目的全球唯一标识符，通常使用全限定的包名区分该项目和其他项目。并且构建时生成的路径也是由此生成， 如com.mycompany.app生成的相对路径为：/com/mycompany/app-->
        <groupId>asia.banseon</groupId>
        <!-- 构件的标识符，它和group ID一起唯一标识一个构件。换句话说，你不能有两个不同的项目拥有同样的artifact ID和groupID；在某个 特定的group ID下，artifact ID也必须是唯一的。构件是项目产生的或使用的一个东西，Maven为项目产生的构件包括：JARs，源 码，二进制发布和WARs等。-->
        <artifactId>banseon-maven2</artifactId>
        <!--项目当前版本，格式为:主版本.次版本.增量版本-限定版本号-->
        <version>1.0-SNAPSHOT</version>
        <!--项目的打包类型，例如jar、war、ear、pom。插件可以创建他们自己的构件类型，所以前面列的不是全部构件类型
            默认是jar
            设置为pom一般是父工程用于统一管理子模块的版本、依赖、插件，本身不写代码
        -->
        <packaging>jar</packaging>


        <!--项目的名称, Maven产生的文档用-->
        <name>banseon-maven</name>
        <!--项目主页的URL, Maven产生的文档用-->
        <url>http://www.baidu.com/banseon</url>
        <!-- 项目的详细描述, Maven 产生的文档用。  当这个元素能够用HTML格式描述时（例如，CDATA中的文本会被解析器忽略，就可以包含HTML标 签）， 不鼓励使用纯文本描述。如果你需要修改产生的web站点的索引页面，你应该修改你自己的索引页文件，而不是调整这里的文档。-->
        <description>A maven project to study maven.</description>


        <!-- 子模块（有时称作子项目） 被构建成项目的一部分。列出的每个模块元素是指向该模块的目录的相对路径-->
        <modules>
            <module>test</module>
        </modules>

        <!-- 通过 properties 标签即可自定义变量配置，然后在其他使用 ${} 引用变量，如下是 ${xxx.version} -->
        <properties>
            <xxx.version>1.4</xxx.version>
        </properties>

        <!-- 当一共项目包含多个模块，且多个模块引用了相同依赖时显然重复引用是不太合适的，而通过 dependencyManagement 即可很好的解决依赖共用的问题。将项目依赖统一定义在父模块的 dependencyManagement 标签中，子模块只需继承父模块并在 dependencies 引入所需的依赖，便可自动读取父模块 dependencyManagement 所指定的版本。dependencyManagement 既不会在当前模块引入依赖，也不会给其子模块引入依赖，但其可以被继承的，只有在子模块下同样声明了该依赖，才会引入到模块中，子模块中只需在依赖中引入 groupId 与 artifactId 即可, 也可以指定版本则会进行覆盖
        -->
        <dependencyManagement>
            <dependencies>
                <!--参见dependencies/dependency元素-->
                <dependency>......</dependency>
            </dependencies>
        </dependencyManagement>


        <!--该元素描述了项目相关的所有依赖。 这些依赖组成了项目构建过程中的一个个环节。它们自动从项目定义的仓库中下载。要获取更多信息，请看项目依赖机制。-->
        <dependencies>
            <!--参见dependencies/dependency元素-->
            <dependency>......</dependency>
        </dependencies>


        <!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。
        默认项目打包后 /resources 目录下文件都将统一打包进编译后的 JAR 文件，通过resource 标签可以设置打包时排除包中的配置文件，启动jar时只需将配置文件放置于 JAR 同级目录，这样方便修改配置文件
        -->
        <resources>
            <!--这个元素描述了项目相关或测试相关的所有资源路径-->
            <resource>
                <!-- 描述了资源的目标路径。该路径相对target/classes目录（例如${project.build.outputDirectory}）。举个例子，如果你想资源在特定的包里(org.apache.maven.messages)，你就必须该元素设置为org/apache/maven/messages。然而，如果你只是想把资源放到源码目录结构里，就不需要该配置。-->
                <targetPath/>
                <!--是否使用参数值代替参数名。参数值取自properties元素或者文件里配置的属性，文件在filters元素里列出。-->
                <filtering/>
                <!--描述存放资源的目录，该路径相对POM路径-->
                <directory/>
                <!--包含的模式列表，例如**/*.xml.-->
                <includes/>
                <!--排除的模式列表，例如**/*.xml-->
                <excludes/>
            </resource>
        </resources>


        <!--构建项目需要的信息-->
        <build>
            <!--构建产生的所有文件存放的目录-->
            <directory/>
            <!--产生的构件的文件名，默认值是${artifactId}-${version}。-->
            <finalName/>
            
            <!--子项目可以引用的默认插件信息。该插件配置项直到被引用时才会被解析或绑定到生命周期。给定插件的任何本地配置都会覆盖这里的配置-->
            <pluginManagement>
                <!--使用的插件列表 。-->
                <plugins>
                    <!--plugin元素包含描述插件所需要的信息。-->
                    <plugin>
                        <!--插件在仓库里的group ID-->
                        <groupId/>
                        <!--插件在仓库里的artifact ID-->
                        <artifactId/>
                        <!--被使用的插件的版本（或版本范围）-->
                        <version/>
                        <!--是否从该插件下载Maven扩展（例如打包和类型处理器），由于性能原因，只有在真需要下载时，该元素才被设置成enabled。-->
                        <extensions/>
                        <!--在构建生命周期中执行一组目标的配置。每个目标可能有不同的配置。-->
                        <executions>
                            <!--execution元素包含了插件执行需要的信息-->
                            <execution>
                                <!--执行目标的标识符，用于标识构建过程中的目标，或者匹配继承过程中需要合并的执行目标-->
                                <id/>
                                <!--绑定了目标的构建生命周期阶段，如果省略，目标会被绑定到源数据里配置的默认阶段-->
                                <phase/>
                                <!--配置的执行目标-->
                                <goals/>
                                <!--配置是否被传播到子POM-->
                                <inherited/>
                                <!--作为DOM对象的配置-->
                                <configuration/>
                            </execution>
                        </executions>
                        <!--项目引入插件所需要的额外依赖-->
                        <dependencies>
                            <!--参见dependencies/dependency元素-->
                            <dependency>......</dependency>
                        </dependencies>
                        <!--任何配置是否被传播到子项目-->
                        <inherited/>
                        <!--作为DOM对象的配置-->
                        <configuration/>
                    </plugin>
                </plugins>
            </pluginManagement>

            <!--使用的插件列表-->
            <plugins>
                <!--参见build/pluginManagement/plugins/plugin元素-->
                <plugin>
                    <groupId/>
                    <artifactId/>
                    <version/>
                    <extensions/>
                    <executions>
                        <execution>
                            <id/>
                            <phase/>
                            <goals/>
                            <inherited/>
                            <configuration/>
                        </execution>
                    </executions>
                    <dependencies>
                        <!--参见dependencies/dependency元素-->
                        <dependency>......</dependency>
                    </dependencies>
                    <goals/>
                    <inherited/>
                    <configuration/>
                </plugin>
            </plugins>
        </build>
</project>
    
~~~ 












