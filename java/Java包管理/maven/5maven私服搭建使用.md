## maven私服搭建（参考：https://www.cainiaoplus.com/nexus/nexus-install.html）
1. 从 https://help.sonatype.com/en/repository-manager-2.html#Download-NexusRepositoryManager2OSS 下载Nexus 2.x （当然这个不是最新的）解压缩可获得 nexus-2.14.20-02 和 sonatype-work 2 个目录，nexus-2.14.20-02目录中包含了 Nexus 2.x 运行所需要的文件，如启动脚本、依赖 jar 包、配置文件等。sonatype-work目录中包含了 Nexus 2.x 生成的配置文件、日志文件等。

2. nexus默认占用的是8081端口，如需修改请编辑 nexus-2.14.20-02\conf\nexus.properties
~~~
application-port=8081
~~~

3. 进入 \nexus-2.14.20-02\bin\jsw 文件夹，根须操作系统版本选择合适的目录，由于我的操作系统是 Windows 10 64 位，所以我选择 windows-x86-64 目录，进入 windows-x86-64 目录后可以如下文件
~~~
console-nexus.bat：启动 Nexus 并在 DOS 命令行中展示启动过程
install-nexus.bat：将 Nexus 安装为 Windows 服务，开机自动启动
start-nexus.bat：启动 Nexus
stops-nexus.bat：停止 Nexus
uninstall-nexus.bat：与 install-nexus.bat 相对应，负责卸载 Nexus 服务
~~~

4. 双击运行 install-nexus.bat 安装 Nexus 服务，然后运行 start-nexus.bat 启动服务。
~~~
运行 install-nexus.bat 安装服务，若提示”wrapper | OpenSCManager failed - 拒绝访问。 (0x5)“，只要关闭窗口，以管理员身份运行即可解决。
运行 start-nexus.bat 启动服务，若提示”wrapper | OpenSCManager failed - 拒绝访问。 (0x5)“，只要关闭窗口，以管理员身份运行即可解决。
~~~

5. 访问 http://localhost:8081/nexus 在登录页面输入用户名和密码，默认分别为：admin 和 admin123

## Nexus仓库使用
Nexus 作为一款 Maven 仓库管理器，仓库（Repository）自然是 Nexus 最核心的概念。Nexus 中提供了许多仓库概念，如代理仓库、宿主仓库以及仓库组等。Nexus 为每一种仓库都提供了丰富的配置参数
在Repositories，可以看到 Nexus 自带的几个内置仓库


Nexus 2.x 默认创建了 6 个仓库，我们称它们为 Nexus 内置仓库。
- Central：该仓库用来代理 Maven 中央仓库，其策略为 Release，只会下载和缓存中央仓库中的发布版本的构件。
- Releases：策略为 Release 的宿主仓库，用来部署公司或组织内部的发布版本构件。
- Snapshots：策略为 Snapshot 的宿主仓库，用来部署公司或组织内部的快照版本构件。
- 3rd party：策略为 Release 的宿主仓库，用来部署第三方发布版本构件，这些构件一般无法从任何远程仓库中获得。
- Public Repositories：该仓库组将上述所有存储策略为 Release 的仓库聚合并通过统一的地址提供服务。

在仓库列表中，每个仓库都具有一系列属性：
- Type：仓库的类型，Nexus 中有 4 中仓库类型：group（仓库组）、hosted（宿主仓库）、proxy（代理仓库）以及 virtual（虚拟仓库）。
- Format：仓库的格式。
- Policy：仓库的策略，表示该仓库是发布（Release）版本仓库还是快照（Snapshot）版本仓库。
- Repository Status：仓库的状态。
- Repository Path：仓库的路径。

Nexus 仓库按照类型（Type）区分，主要分为以下 3 个类型：
- 代理仓库（proxy）：用来代理远程公共仓库，如 Maven 中央仓库、JBoss 远程仓库，从代理仓库中下载构件，代理仓库会从远程仓库下载并缓存构件
- 宿主仓库（hosted）：又称 Nexus 本地仓库，该仓库通常用来部署本地项目所产生的jar包构件
- 仓库组（group）：用来聚合代理仓库和宿主仓库，为这些仓库提供统一的服务地址，以便 Maven 可以更加方便地获得这些仓库中的构件，从仓库组中下载构件，仓库组会从其包含的宿主仓库和代理仓库中获取构件，主要是用于方便客户端操作获取构件

上面讲了一大堆其实就是推荐你上传使用 Releases 和 Snapshots 仓库，下载直接使用Public Repositories 仓库
当然也可以自己手动的创建一个hosted仓库，然后编辑Public Repositories 仓库配置将手动创建的仓库加入其中

### 使用 Maven 上传构件到 Nexus
参考：https://www.cainiaoplus.com/nexus/nexus-deploy.html 里面有maven上传构件和手动打包构件上传到Nexus
1. 配置项目的 pom.xml 文件
~~~xml
<!-- 这里使用的是默认内置的 Releases和Snapshots仓库

    distributionManagement 元素：负责将指定的构件部署到 Nexus 指定的仓库中。

    repository 元素：distributionManagement 的子元素，用于定义部署 Release 版本的构件被部署的仓库。它有 2 子元素：id 和 url ，其中 id 为 Nexus 仓库的唯一标识，url 为 Nexus 宿主仓库的 url。

    snapshotRepository 元素：distributionManagement 的子元素，用于定义部署 Snapshot 版本的构件被部署的仓库。它也有 2 子元素：id 和 url ，且与 repository 中含义一致。

    这些都可以在Nexus找到对应的配置
 -->
<project>
    <distributionManagement>
        <repository>
            <id>releases</id>
            <url>http://localhost:8081/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
        </snapshotRepository>
    </distributionManagement>
</project>
~~~


2. 在 setting.xml 中配置认证信息
最初版本的 Nexus 没有为宿主仓库提供任何的安全措施。如果宿主仓库开启了部署功能，任何人可以连接并部署构件至这个仓库，这显然是极不安全的。因此，现在的 Nexus 中增加了权限认证，Nexus 对于匿名用户是只读的，若想部署构件到 Nexus 中，则需要在 setting.xml 中配置如下认证信息
~~~xml
<!-- 注意：server 元素中的 id 必须和 pom.xml 中 distributionManagement 元素对应仓库的 id 保持一致。Maven 在部署构件时，会先根据 id 查找用户名称和密码进行认证和登录，然后将构件部署到对应得宿主仓库 -->
<settings>
    <servers>
        <server>
            <id>releases</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
        <server>
            <id>snapshots</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>
</settings>
~~~


3. 使用 mvn 命令部署构件
~~~shell
# 完成打包上传构件到Nexus服务器，可以在对应仓库下找到打包的构件（一般都是jar）
mvn clean deploy

# <version>1.0.0-SNAPSHOT</version> 结尾带-SNAPSHOT的版本会上传到snapshots仓库
# 其他的像 1.0.0 这样的版本会上传到releases仓库
~~~


### 从Nexus下载构件依赖
在引用依赖时会先从私人仓库进行读取，如果未找到再从中央仓库下载至私人仓库，最后再下载到本地仓库
1. 在 setting.xml 中配置添加镜像
~~~xml
<!-- 这里使用的是默认内置的 Public Repositories 仓库 -->
<mirrors>
    <mirror>
        <id>nexus</id>
        <name>nexus name</name>
        <mirrorOf>*</mirrorOf>
        <url>http://localhost:8081/nexus/content/groups/public</url>
    </mirror>
</mirrors>
~~~

2. 在 setting.xml 中配置认证信息，如果Nexus支持匿名访问可以不用配置认证信息
~~~xml
<!-- 注意：server 元素中的 id 必须和上述配置的镜像的 id 保持一致。-->
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>
</settings>
~~~

3. 然后就可以在项目中引入Nexus私服中的依赖使用了
~~~xml
<dependency>
    <groupId>test</groupId>
    <artifactId>com-cli-test</artifactId>
    <version>1.7.0</version>
</dependency>
~~~





