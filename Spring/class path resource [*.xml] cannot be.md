## SSM-class path resource [*.xml] cannot be 

* class path resource [applicationContext.xml] cannot be opened because it does not exist

* [reference website](http://blog.csdn.net/grwgwef/article/details/48521309)
###solve  way is add   		 **`*`**  		  after `classpath`


source code:
```java
	<servlet>
		<servlet-name>crm</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<!-- 此处不配置 默认找 /WEB-INF/[servlet-name]-servlet.xml -->
      
			<param-value>classpath:springmvc.xml</param-value>
      
      
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
```

change code:
			<param-value>classpath*:springmvc.xml</param-value>
