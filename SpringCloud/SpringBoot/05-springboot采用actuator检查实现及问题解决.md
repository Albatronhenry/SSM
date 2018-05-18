[springboot采用actuator检查实现及问题解决](https://blog.csdn.net/u013076044/article/details/60780151)
-----------------

* 在之前文章的基础上,我们实现spring的监控:
   * 只需在相应的springboot项目的pom.xml文件中添加
```xml
    <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

```

然后启动项目即可;

* 可能遇到问题(访问/beans  /evn 等敏感的信息时候报错):

     Whitelabel Error Page
  
     This application has no explicit mapping for /error, so you are seeing this as a fallback.
     
     Tue Apr 17 14:01:49 CST 2018
     
    ` There was an unexpected error (type=Unauthorized, status=401). `
    
     Full authentication is required to access this resource. 
     
     * 问题解决方法2种:
       * 都是在application.properties添加配置参数
         * ①  management.security.enabled=false   
         * ② endpoints.sensitive=false   endpoints.[name].sensitive=false
         
         
