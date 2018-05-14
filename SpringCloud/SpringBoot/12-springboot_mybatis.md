[springboot_mybatis.md](https://blog.csdn.net/u011320740/article/details/79256807)
---------------
* pom.xml文件需要添加的依赖
```xml
<dependencies>  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-web</artifactId>  
        </dependency>  
  
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-test</artifactId>  
            <scope>test</scope>  
        </dependency>  
          
        <dependency>  
            <groupId>org.mybatis.spring.boot</groupId>  
            <artifactId>mybatis-spring-boot-starter</artifactId>  
            <version>1.3.0</version>  
        </dependency>  
          
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <scope>runtime</scope>  
        </dependency>  
        <dependency>  
            <groupId>com.alibaba</groupId>  
            <artifactId>druid</artifactId>  
            <version>1.0.29</version>  
        </dependency>  
    </dependencies>  
```
* application.properties
```properties
spring.datasource.url=jdbc\:mysql\://127.0.0.1\:3306/springboot_mybatis?useUnicode\=true&characterEncoding\=gbk&zeroDateTimeBehavior\=convertToNull  
spring.datasource.username=root  
spring.datasource.password=root111  
spring.datasource.driver-class-name=com.mysql.jdbc.Driver  
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource  
spring.datasource.initialSize=5    
spring.datasource.minIdle=5    
spring.datasource.maxActive=20    
spring.datasource.maxWait=60000    
mybatis.mapper-locations=classpath*:mapper/*Mapper.xml  
mybatis.type-aliases-package=com.yonyou.mybatis.entity  
```
* UserMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.yonyou.mybatismapper.UserMapper">  
   
  <select id="findUserInfo" resultType="com.yonyou.mybatisentity.User">  
    select id,name  from user;  
  </select>  
  <insert id="addUserInfo" parameterType="com.yonyou.mybatisentity.User">  
    insert into user (id, name  
      )  
    values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}  
      )  
  </insert>  
  <delete id="delUserInfo" parameterType="java.lang.Integer">  
   delete from user where id = #{id,jdbcType=INTEGER}  
  </delete>  
   
</mapper>  
```
* UserMapper.java
```java
import org.apache.ibatis.annotations.Mapper; 
@Mapper  
public interface UserMapper {  
  
    public List<User> findUserInfo();  
    public int addUserInfo(User user);  
    public int delUserInfo(int id);  
}  
```
* User.java
```java
public class User implements Serializable {    
    private Long id;    
    private String name;        
   //getter/setter省略    
    @Override    
    public boolean equals(Object o) {    
        if (this == o) return true;    
        if (o == null || getClass() != o.getClass()) return false;    
    
        User user = (User) o;    
    
        if (id != null ? !id.equals(user.id) : user.id != null) return false;    
    
        return true;    
    }    
    
    @Override    
    public int hashCode() {    
        return id != null ? id.hashCode() : 0;    
    }    
}    
```
* UserService.java
```java
public interface UserService {  
    public List<User> getUserInfo();  
      
    public void insert(User user);  
} 
```
* UserServiceImpl.java
```java
@Service  
public class UserServiceImpl implements UserService{  
  
    @Autowired  
    private UserMapper userMapper;  
  
    public List<User> getUserInfo(){  
        return userMapper.findUserInfo();  
    }  
  
      
    public void insert(User user) {  
        userMapper.addUserInfo(user);  
          
    }  
}  
```
* UserController.java
```java
@RestController    
@RequestMapping("/user")    
public class UserController {    
    @Autowired  
    private UserService userService;  
      
    @RequestMapping("/getUserInfo")  
    public List<User> getUserInfo() {  
        List<User> user = userService.getUserInfo();  
        System.out.println(user.toString());  
        return user;  
    }  
      
    @RequestMapping("/addUserInfo")  
    public String addUserInfo() {  
        User user = new User();  
        user.setId(3L);  
        user.setName("cwh");  
        userService.insert(user);  
        return "success:"+user.toString();  
    }  
      
      
}    
```
然后运行就可以了
