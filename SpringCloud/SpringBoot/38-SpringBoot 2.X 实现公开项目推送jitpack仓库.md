### SpringBoot 2.X 实现公开项目推送jitpack仓库并在线编译

1. 自行创建一个公有项目  **com.gitee.albatron-henry/JitPackTest**

   

2. https://kitpack.io 输入仓库名称,点击Look up,即可根据不同显示不同的版本，点击 getit即可一键构建对应项目maven仓库项目

![image-20210331185202089](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210331185202089.png)



​    3.其余项目中，引用以上构建项目，只需在pom文件中进行如下依赖即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!-- 1.依赖相关仓库项目 -->
        <dependency>
            <groupId>com.gitee.albatron-henry</groupId>
            <artifactId>JitPackTest</artifactId>
            <!-- 1.1 对应构建的版本 -->
            <version>1.0.1</version>
        </dependency>
    </dependencies>
	<!-- 2.添加对应jitpack仓库地址 -->
    <repositories>
        <repository>
            <id>jitpack.io</id>
            <url>https://jitpack.io</url>
        </repository>
    </repositories>

</project>

```

