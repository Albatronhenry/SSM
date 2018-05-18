## [springboot_mybatis_pagehelper主要参考](http://412887952-qq-com.iteye.com/blog/2303121)
[springboot_mybatis_pagehelper插件使用参考](https://github.com/abel533/MyBatis-Spring-Boot)
--------
* 新建maven project
```xml
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
    <!-- 1.3.0这个版本的可以注入@MapperScan注解,1.3.4好像注入不了 -->
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.0</version>
		</dependency>
		<!--普通类型分页依赖  -->
		<!-- <dependency> <groupId>com.github.pagehelper</groupId> <artifactId>pagehelper</artifactId> 
			<version>4.1.0</version> </dependency> -->
		<!--mapper -->
		<!--springboot pagehelper 依赖 -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper-spring-boot-starter</artifactId>
			<version>1.2.3</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
	</dependencies>
```
  * 配置mysql
```properties
###datasource
spring.datasource.url = jdbc:mysql://localhost:3306/springboot_mybatis
spring.datasource.username = root
spring.datasource.password = root111
spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.max-active=20
spring.datasource.max-idle=8
spring.datasource.min-idle=8
spring.datasource.initial-size=10
```
  * Application启动类添加 @MapperScan("com.yonyou.*.mapper")  
  * 创建一个User实体类(ID name 和get/set方法),代码省略
  * 创建UserMapper
  ```java
  public interface UserMapper {
	@Select("select *from User where name = #{name}")
	public List<User> likeName(String name);

	@Select("select *from User where id = #{id}")
	public User getById(long id);

	@Select("select name from User where id = #{id}")
	public String getNameById(long id);
}
```
* 创建UserService
```java
@Service
public class UserService {
	@Autowired
	private UserMapper userMapper;

	public List<User> likeName(String name) {
		return userMapper.likeName(name);

	}
}
```
* 创建UserController
```java
@RestController
public class UserController {
	@Autowired
	private UserService userService;
	 @RequestMapping("/likeName")
	public List<User> likeName(String name) {
  //下面这一行代码就是实现分页的,第一个参数是第几页；第二个参数是每页显示条数。
		 PageHelper.startPage(1,1);
     
		return userService.likeName(name);

	}
}
```
* 要使用上面的分页代码,还需要添加一个mybatis pagehelper配置类,注册MyBatis分页插件PageHelper
```java
//注册MyBatis分页插件PageHelper
@Configuration
public class MyBatisConfiguration {
	@Bean
	public PageHelper pageHelper() {
		System.err.println("MyBatisConfiguration.pageHelper()");
		PageHelper pageHelper = new PageHelper();
		Properties p = new Properties();
		p.setProperty("offsetAsPageNum", "true");
		p.setProperty("rowBoundsWithCount", "true");
		p.setProperty("reasonable", "true");
		pageHelper.setProperties(p);
		return pageHelper;
	}
}
```
* 最后就可以启动,http://127.0.0.1:8080/likeName?name=henry 进行访问了,会发现如果controller类中没加下面这一行代码,就会全部显示,加了就会分页
```java
PageHelper.startPage(1,1);
```
  
