# Maven是什么
pache Maven 是一个基于 POM（Project Object Model） 的 项目管理和构建工具，主要用于 Java 项目（也支持其他语言）
1. 项目管理工具：自动从仓库下载和管理项目需要的 JAR 文件。解决版本冲突和传递性依赖
2. 构建工具：编译、测试、打包和部署

# Maven安装
1. 官网下载地址：https://maven.apache.org/download.cgi 选择对应版本的二进制包下载解压缩
2. 配置java环境变量(如果已经有java环境可以跳过这一步配置)：新增JAVA_HOME环境变量，值为 jdk 解压路径，同时编辑 Path 环境变量，添加 %JAVA_HOME%\bin
3. 配置maven环境变量：新增MAVEN_HOME环境变量，值为 Maven 解压路径，同时编辑 Path 环境变量，添加 %MAVEN_HOME%\bin
4. 验证安装：新开cmd窗口执行 mvn -v 可以查看安装的Maven版本

