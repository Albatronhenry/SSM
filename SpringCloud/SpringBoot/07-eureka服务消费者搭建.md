[eureka服务消费者搭建]()
--------------

* 1 eureka搭建:
-------------
  * 创建相应maven eureka项目,然后Application类中添加注解
```java  
  @SpringBootApplication
  @EnableEurekaServer
``` 
  * yml文件添加
```yml
server:
  port: 9000 #当前应用端口
spring:
  application:
    name: eureka-registry #应用名称
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:9000/eureka/  #Eureka Server地址
    register-with-eureka:
      false #因为自己是注册中心，所以不注册自己
    fetch-registry:
      false
  instance:
    metadataMap:
      instanceId: ${spring.application.name}:${spring.application.instance_id:${random.value}}
  
```
  * 可以启动,登陆看到相应页面了
  
  
* 2 eureka client搭建
-----------------
  * 创建相应maven eureka项目,然后Application类中添加注解
```java
  @EnableDiscoveryClient 
  @SpringBootApplication
```
  * 新建相应的controller类
```java
  @RestController
public class TestControl  {
	@RequestMapping("/henrytest")
	public String testHello(@RequestParam String name) {
		return "Hello world," + name;
	}

}
```
  * 可以启动,登陆看到相应页面了,同时刷新eureka页面可以看到client加入进去了
  
  
* 3  eureka server 搭建
------------------
  * 创建相应maven feign项目,然后Application类中添加注解
  ```java
  @EnableDiscoveryClient  //启用服务注册与发现
  @EnableFeignClients     //启用feign进行远程调用
  @SpringBootApplication
  ```
  
  * 创建相应的接口类
  ```java
  @FeignClient(name= "eureka.client") //此处name为里面的名称
  public interface HelloRemote {
    @RequestMapping(value ="/henrytest")
    public String testHello(@RequestParam(value = "name") String name);
  }
  ```
  * 创建相应controller类-->
  * 注意: ` name:远程服务名，及spring.application.name配置的名称,此类中的方法和远程服务中contoller中的方法名和参数需保持一致。 `
  ```java
  @RestController
public class ConsumerController {

    @Autowired
    HelloRemote helloRemote;

    @RequestMapping("/henrytest/{name}")
    public String index(@PathVariable("name") String name) {
        return helloRemote.testHello(name);
    }
}
```

*依次启动eureka、eureka-client、eureka-server对应的三个项目

先输入：` http://localhost:9080/henrytest?name=hen ` 检查eureka-client服务是否正常

返回：Hello world,hen

说明eureka-client正常启动，提供的服务也正常。

浏览器中输入：` http://localhost:9001/henrytest/hen `

返回：Hello world,hen

说明客户端已经成功的通过feign调用了远程服务hello，并且将结果返回到了浏览器。

[reference_website1](https://blog.csdn.net/ityouknow/article/details/72085662)

[reference_website1](https://blog.csdn.net/ityouknow/article/details/72235670)
