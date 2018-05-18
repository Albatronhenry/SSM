[springboot核心注解](https://blog.csdn.net/wangb_java/article/details/70943500?utm_source=itdadao&utm_medium=referral)
---------------

![@SpringBootApplication](https://github.com/Albatronhenry/UploadFile/blob/master/pic/springbootAnotaion.png)

其中

    @SpringBootApplication 为Application类中注解
    
```java

package com.yonyou.helloworld;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloworldApplication {

	public static void main(String[] args) {
		SpringApplication.run(HelloworldApplication.class, args);
	}
}


```

    @RestController 为对应controller类中注解
    
```java

package com.yonyou.helloworld.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloworldController {
	
	@Value("${henry.msg}")
	private String msg;
	
	@RequestMapping("/henry")
     public String SayHello(){
    	 return "hello , "+msg;
     }
}


```

脑图链接地址:http://naotu.baidu.com/file/5d297a73dbcc9a364e74b09f25026565
