添加依赖，参考 https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E

~~~xml
    <!--<parent>    当前项目继承的父maven项目，一般是自己公司的项目，如果是开源项目一般放入dependencyManagement中管理，如下面的dependencyManagement       -->  
    <!--    <groupId>org.springframework.boot</groupId>-->
    <!--    <artifactId>spring-boot-starter-parent</artifactId>-->
    <!--    <version>2.3.2.RELEASE</version>-->
    <!--    <relativePath/>-->
    <!--</parent>-->
    <groupId>com.eric.springcloud</groupId>
    <artifactId>alibaba-parent-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>parent-demo</name>
    <description>parent-demo project for Spring Cloud alibaba</description>
    <modules>
        <module>order</module>
        <module>stock</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
    </properties>
    
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>     <!--该标签中的依赖不会被子项目自动继承，如果子项目需要其中的依赖需要显示声明需要依赖-->
    <dependencies>         
        <!-- Spring Cloud Alibaba的版本管理-->
        <dependency>       
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- Spring Boot的版本管理-->
        <dependency>       <!--替换了上面parent标签中继承父maven项目-->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.3.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- Spring cloud 的版本管理-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR9</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
    
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
~~~