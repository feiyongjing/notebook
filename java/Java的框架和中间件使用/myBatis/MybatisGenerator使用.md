### 官方文档地址 http://www.mybatis.org/generator/

### 官方工程地址 https://github.com/mybatis/generator/releases
---

## 使用Maven插件

1. 引用的Jar包，注意Mysql也是要提前引用的

~~~xml
<build>
    <plugins>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.4.0</version>
            <dependencies> <!-- 有其他的插件使用需要在这里引入依赖-->
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>8.0.18</version>
                </dependency>
            </dependencies>
            <configuration>
                <overwrite>true</overwrite>
            </configuration>
        </plugin>
    </plugins>
</build>
~~~

2. 在resources目录下创建generatorConfig.xml文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="wxshop" targetRuntime="MyBatis3"> <!-- 设置MyBatis的版本 -->
        <!-- 引用的插件 --> 
        <!-- 这个插件给自动生成的TO类 implements Serializable-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
        <!-- 这个插件给自动生成的Mapper类 添加 @Mapper注解 -->
        <plugin type="org.mybatis.generator.plugins.MapperAnnotationPlugin" />
        <!-- 这个插件给自动生成的EXample类 添加SQL分页方法，注意在mybatis-generator-core的1.4.0版本无法使用，在1.3.7可以正常使用-->
        <plugin type="com.qiukeke.mybatis.plugins.MySQLLimitPlugin" />
        <!-- 这个插件给自动生成的Mapper.xml文件 每次重复生成文件时覆盖原文件而不是追加到原XML文件的末尾 -->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
        <!--数据库连接配置-->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/webmvc?useSSL=false&allowPublicKeyRetrieval=true"
                        userId="root"
                        password="root">
              <!-- mysql不支持 schema 会在生成实体类是生成myslq默认数据库其他表映射的类，添加该配置后可以消除该问题 -->
              <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="true" />
        </javaTypeResolver>

        <!-- 配置生成的实体类路径 -->
        <javaModelGenerator targetPackage="com.feiyongjing.generate" targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- 配置sql语句的xml文件路径 -->
        <sqlMapGenerator targetPackage="db.mybatis"  targetProject="src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- 配置Mapper接口的路径 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.feiyongjing.generate"  targetProject="src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        
        <!-- 配置实体类来自那个数据库和那张表，设置生成的实体类名和其他配置 -->
        <table schema="webmvc" tableName="USER" domainObjectName="User" >
            <!-- 是否使用实际的列名 -->
            <property name="useActualColumnNames" value="false"/>
            <!-- 设置主键，与使用的Sql语言 -->
            <generatedKey column="ID" sqlStatement="MySql" identity="true" />
        </table>
    </context>
</generatorConfiguration>
~~~

3. 最后运行命令生成文件

~~~shell
mvn mybatis-generator:generate
~~~