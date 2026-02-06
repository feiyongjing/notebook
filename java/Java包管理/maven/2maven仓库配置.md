
## Maven仓库
Maven仓库中有jar依赖包，而Maven仓库分为本地仓库、私有仓库、远程仓库
- 本地仓库是指在本机电脑的一个目录下存放依赖jar包
- 私有仓库是指在个人或者组织搭建Maven仓库存储jar包，一般情况下都是在局域网内部，仅供个人或者组织内部人员使用。私有仓库的好处是局域网内下载传输jar包快（一些远程仓库常用的jar包可以先下载到私有仓库方便后续其他人下载这些jar包），并且存储个人或者组织研发的jar包避免泄漏
- 远程仓库是指公网上供所有人下载开源jar包的公共Maven仓库

### 本地仓库配置
maven的本地仓库默认位于~/.m2，下载的第三⽅包放在这⾥进⾏缓存，不必下次需要的时候在从中央仓库下载
可以修改本地仓库的路径
1. 创建一个目录存放本地仓库，我比较喜欢直接存放在maven的安装目录下，找到之前配置的环境变量MAVEN_HOME目录，新建一个maven_repository文件夹作为本地的仓库
2. maven安装目录下的conf目录中有settings.xml配置，编辑这个文件找到localRepository标签
~~~xml
<!-- localRepository标签中配置本地仓库地址 -->
<localRepository>D:/maven/maven_repository</localRepository>
~~~

### 镜像仓库加速配置
Maven中央仓库手动下载jar包：https://repo1.maven.org/maven2/
远程仓库无需进行配置，不过Maven中央仓库在国外，下载依赖包比较慢，可以配置国内的镜像仓库进行加速
maven安装目录下的conf目录中有settings.xml配置，编辑这个文件找到mirrors标签在内部添加mirror镜像仓库配置
~~~xml
<mirrors>
    <!-- 阿里云镜像central库 -->
    <!-- central Maven中央库（默认仓库），加速访问Java标准库（如commons-*、log4j等），最常用的公共依赖仓库，包含绝大多数开源Java库 -->
    <mirror>
        <id>aliYunMaven</id>
        <name>aliyun maven</name>
        <mirrorOf>central</mirrorOf>
        <url>https://maven.aliyun.com/repository/central</url>
    </mirror>

    <!-- 阿里云镜像public库 -->
    <!-- public 同时代理两个仓库的依赖：central + jcenter聚合仓，简化配置，一次引用多个源（但可能增加依赖冲突风险）     -->
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>*</mirrorOf>
        <name>阿里云公共仓库</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>

    <!--清华大学镜像 -->
    <mirror>
        <id>tsinghuaUniversityMaven</id>
        <name>tsinghuaUniversity Maven</name>
        <mirrorOf>external:http:*</mirrorOf>
        <url>https://repo.maven.apache.org/maven2/</url>
    </mirror>

    <!--华为镜像 -->
    <mirror>
        <id>huaWeiMaven</id>
        <name>huaWei Maven</name>
        <mirrorOf>external:http:*</mirrorOf>
        <url>https://repo.huaweicloud.com/repository/maven/</url>
    </mirror>
</mirrors>
~~~
