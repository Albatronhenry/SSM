[整合Enreka](http://xujin.org/sc/sc-eureka-01/)
整合Enreka中@EnableEurekaServer无法解决，需要注意一下pom.xml中：
```xml
<dependencies>

        <!-- eureka 监控组件，需要添加下面的springcloud依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency> 
 </dependencies>
 
 <!-- 引入spring cloud的依赖,依赖一定要对，不然注解加不进来 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <!--<version>Finchley.RC1</version>-->
                <!--<version>Edgware.SR4</version>-->
                <!--此处特别注意对应version，不然springcloud依赖不进来，本次使用的springboot版本2.1.1-->
                <version>Camden.SR5</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
