[spring_jpa_thymeleaf](http://www.ityouknow.com/springboot/2017/09/23/spring-boot-jpa-thymeleaf-curd.html)
-------------
* 1.创建maven (jpa web thymeleaf mysql)项目
* 2.配置文件添加:
```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=true
spring.datasource.username=root
spring.datasource.password=root111
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.properties.hibernate.hbm2ddl.auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.show-sql= true

spring.thymeleaf.cache=false
```
其中 ` propertiesspring.thymeleaf.cache=false `是关闭thymeleaf的缓存，不然在开发过程中修改页面不会立刻生效需要重启，生产可配置为true。

在项目resources目录下会有两个文件夹：static目录用于放置网站的静态内容如css、js、图片；templates目录用于放置项目使用的html页面模板

* 3.启动类需要添加` Servlet `的支持

(继承` SpringBootServletInitializer ` 覆盖` configure() ` 把启动类` SpringbootJpaThymeleafApplication ` 注册进去。

外部web应用服务器构建Web Application Context的时候，会把启动类添加进去。)
```java
package com.yonyou.spring_j_t;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class SpringbootJpaThymeleafApplication extends SpringBootServletInitializer{
     @Override
     protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
         return application.sources(SpringbootJpaThymeleafApplication.class);
     }
	public static void main(String[] args) {
		SpringApplication.run(SpringbootJpaThymeleafApplication.class, args);
	}
}
```
* 4.数据库层实体类映射数据库表
```java
@Entity
public class User {
    @Id
    @GeneratedValue
    private long id;
    @Column(nullable = false, unique = true)
    private String userName;
    @Column(nullable = false)
    private String password;
    @Column(nullable = false)
    private int age;
    //   getter/setter  无参/有参构造函数均省略
}

```
* 5.继承JpaRepository类会自动实现很多内置的方法，包括增删改查。也可以根据方法名来自动生成相关sql
```java
public interface UserRepository extends JpaRepository<User, Long> {
    User findById(long id);
    Long deleteById(Long id);
}
```
* 6.service调用jpa实现相关的增删改查，实际项目中service层处理具体的业务代码
```java
@Service
public class UserServiceImpl implements UserService{

    @Autowired
    private UserRepository userRepository;

    @Override
    public List<User> getUserList() {
        return userRepository.findAll();
    }

    @Override
    public User findUserById(long id) {
        return userRepository.findById(id);
    }

    @Override
    public void save(User user) {
        userRepository.save(user);
    }

    @Override
    public void edit(User user) {
        userRepository.save(user);
    }

    @Override
    public void delete(long id) {
        userRepository.delete(id);
    }
}
```
* 7.Controller负责接收请求，处理完后将页面内容返回给前端
```java
@Controller
public class UserController {

    @Resource
    UserService userService;


    @RequestMapping("/")
    public String index() {
        return "redirect:/list";
    }

    @RequestMapping("/list")
    public String list(Model model) {
        List<User> users=userService.getUserList();
        model.addAttribute("users", users);
        return "user/list";
    }

    @RequestMapping("/toAdd")
    public String toAdd() {
        return "user/userAdd";
    }

    @RequestMapping("/add")
    public String add(User user) {
        userService.save(user);
        return "redirect:/list";
    }

    @RequestMapping("/toEdit")
    public String toEdit(Model model,Long id) {
        User user=userService.findUserById(id);
        model.addAttribute("user", user);
        return "user/userEdit";
    }

    @RequestMapping("/edit")
    public String edit(User user) {
        userService.edit(user);
        return "redirect:/list";
    }


    @RequestMapping("/delete")
    public String delete(Long id) {
        userService.delete(id);
        return "redirect:/list";
    }
}
```
* 8.html页面
  *list.html
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>userList</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}"></link>
</head>
<body class="container">
<br/>
<h1>用户列表</h1>
<br/><br/>
<div class="with:80%">
    <table class="table table-hover">
        <thead>
        <tr>
            <th></th>
            <th>User Name</th>
            <th>Password</th>
            <th>Age</th>
            <th>Edit</th>
            <th>Delete</th>
        </tr>
        </thead>
        <tbody>
        <tr  th:each="user : ${users}">
            <th scope="row" th:text="${user.id}">1</th>
            <td th:text="${user.userName}">neo</td>
            <td th:text="${user.password}">Otto</td>
            <td th:text="${user.age}">6</td>
            <td><a th:href="@{/toEdit(id=${user.id})}">edit</a></td>
            <td><a th:href="@{/delete(id=${user.id})}">delete</a></td>
        </tr>
        </tbody>
    </table>
</div>
<div class="form-group">
    <div class="col-sm-2 control-label">
        <a href="/toAdd" th:href="@{/toAdd}" class="btn btn-info">add</a>
    </div>
</div>

</body>
</html>
```
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>user</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}"></link>
</head>
<body class="container">
<br/>
<h1>修改用户</h1>
<br/><br/>
<div class="with:80%">
    <form class="form-horizontal"   th:action="@{/edit}" th:object="${user}"  method="post">
        <input type="hidden" name="id" th:value="*{id}" />
        <div class="form-group">
            <label for="userName" class="col-sm-2 control-label">userName</label>
            <div class="col-sm-10">
                <input type="text" class="form-control" name="userName"  id="userName" th:value="*{userName}" placeholder="userName"/>
            </div>
        </div>
        <div class="form-group">
            <label for="password" class="col-sm-2 control-label" >Password</label>
            <div class="col-sm-10">
                <input type="password" class="form-control" name="password" id="password"  th:value="*{password}" placeholder="Password"/>
            </div>
        </div>
        <div class="form-group">
            <label for="age" class="col-sm-2 control-label">age</label>
            <div class="col-sm-10">
                <input type="text" class="form-control" name="age"  id="age" th:value="*{age}" placeholder="age"/>
            </div>
        </div>
        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-10">
                <input type="submit" value="Submit" class="btn btn-info" />
                &nbsp; &nbsp; &nbsp;
                <a href="/toAdd" th:href="@{/list}" class="btn btn-info">Back</a>
            </div>

        </div>
    </form>
</div>
</body>
</html>
```
* 9.效果图
![spring_j_t1](https://github.com/Albatronhenry/UploadFile/blob/master/pic/spring_j_t1.png)
![spring_j_t2](https://github.com/Albatronhenry/UploadFile/blob/master/pic/spring_j_t2.png)
