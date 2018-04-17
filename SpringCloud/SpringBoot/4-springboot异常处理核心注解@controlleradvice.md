[springboot异常处理核心注解@controlleradvice.md]()
-----------

```java

@RestController
public class HelloworldController {

	@Value("${henry.msg}")
	private String msg;

	@RequestMapping("/henry")
	public String SayHello() {
		int hen = 100 / 0;  //此处添加一个异常点
		return "hello , " + msg;
	}
}

```

下面为定义的一个异常测试类:

```java

package com.yonyou.helloworld.exception;

import java.util.HashMap;
import java.util.Map;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

/*
 * 全局作用于@requestmapping上的异常处理
 */
@ControllerAdvice
public class MyControllerException {
		/*返回的是json,所以要加@ResponseBody这个注解,不加的话运行时会报
	     *Check your ViewResolver setup! 
	     *(Hint: This may be the result of an unspecified view, due to default view name generation
	     */
	   @ResponseBody 
	   @ExceptionHandler(value=Exception.class)
       public Map<String,Object> myException(Exception ex){
    	   Map<String,Object> map = new HashMap<String, Object>();
    	   map.put("code", -1);
    	   map.put("msg", ex.getMessage());
    	   
    	   return map;
       }
}


```

页面返回结果值为:

` {"msg":"/ by zero","code":-1}  `


------------------


下面定义一个自定义异常类:

```java

package com.yonyou.helloworld.exception;

public class CustomerException extends RuntimeException {
		private int code;
		private String msg;
		public CustomerException(int code, String msg) {
			super();
			this.code = code;
			this.msg = msg;
		}
//getter/setter方法省略

```

myControllerException中添加如下方法:

```java

 @ResponseBody 
	   @ExceptionHandler(value=CustomerException.class)
	   public Map<String,Object> customerException(CustomerException ex){
		   Map<String,Object> map = new HashMap<String, Object>();
		   map.put("code", ex.getCode());
		   map.put("msg", ex.getMsg());
		   
		   return map;
	   }

```
controller类中直接抛出该异常

```java

@RequestMapping("/henry")
	public String SayHello() {
		throw new CustomerException(-2, "yonyouexception test!!!");
	}
  
```

运行结果:

` {"msg":"yonyouexception test!!!","code":-2} `
