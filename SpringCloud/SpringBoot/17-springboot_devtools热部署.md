[springboot_devtools热部署](http://412887952-qq-com.iteye.com/blog/2300313)
--------------
* 本热部署针对类文件的改变,如果是resource下的static/templates静态资源文件application.properties文件中配置spring.thymeleaf.cache=false即可
  * (前提pom文件中引用了thymeleaf)
* 1.pom.xml中
```xml
       <!--  
           devtools可以实现页面热部署（即页面修改后会立即生效，这个可以直接在application.properties文件中配置spring.thymeleaf.cache=false来实现），      
           实现类文件热部署（类文件修改后不会立即生效），实现对属性文件的热部署。   
           即devtools会监听classpath下的文件变动，并且会立即重启应用（发生在保存时机），注意：因为其采用的虚拟机机制，该项重启是很快的      
        -->  
       <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-devtools</artifactId>  
            <optional>true</optional>  
        </dependency>  
```      
* 2.重要一点:仅仅加入devtools在我们的eclipse中还不起作用，这时候还需要添加的spring-boot-maven-plugin：
```xml
<build>  
       <plugins>  
           <!--  
             用于将应用打成可直接运行的jar（该jar就是用于生产环境中的jar） 值得注意的是，如果没有引用spring-boot-starter-parent做parent，  
                       且采用了上述的第二种方式，这里也要做出相应的改动  
             -->  
            <plugin>  
                <groupId>org.springframework.boot</groupId>  
                <artifactId>spring-boot-maven-plugin</artifactId>  
                <configuration>  
                   <!--fork :  如果没有该项配置，肯呢个devtools不会起作用，即应用不会restart -->  
                    <fork>true</fork>  
                </configuration>  
            </plugin>  
       </plugins>  
   </build>  
```
* 其中加入的是:
```xml
               <configuration>  
                   <!--fork :  如果没有该项配置，肯呢个devtools不会起作用，即应用不会restart -->  
                    <fork>true</fork>  
                </configuration>  
 ```               

[特别注意](https://www.cnblogs.com/lspz/p/6832358.html)使用IDEA，还需配置idea：

（1）File-Settings-Compiler-Build Project automatically

（2）ctrl + shift + alt + /,选择Registry,勾上 Compiler autoMake allow when app running
