[springboot-slf4j输出日志](https://blog.csdn.net/liumiaocn/article/details/53523546)
----------------------
* 创建maven项目,基本就可以直接使用slf4j了
* 创建logback.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml" />
    <logger name="org.springframework.web" level="INFO"/>
    <logger name="org.springboot.sample" level="TRACE" />
    <springProfile name="dev">
        <logger name="org.springboot.sample" level="DEBUG" />
    </springProfile>

    <springProfile name="staging">
        <logger name="org.springboot.sample" level="INFO" />
    </springProfile>
</configuration>
```
* 在启动类中做测试
```java
@RestController
@SpringBootApplication
public class SpringBootImagecodeApplication {
	private final Logger logger = LoggerFactory.getLogger(this.getClass());
	 @RequestMapping("/slf4j")
	  public void greeting(@RequestParam(value="name",defaultValue = "henry") String name){
	    logger.info("## Info  Information ##: {}", name);
	    logger.warn("## Warn  Information ##: {}", name);
	    logger.error("## Error Information ##: {}", name);
	   
	  }
	public static void main(String[] args) {
		SpringApplication.run(SpringBootImagecodeApplication.class, args);
	}
}
```
* 访问http://localhost:8080/slf4j,就可以在控制台看到输出信息了
