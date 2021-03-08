### SpringBoot 2.X 配置文件加载顺序及实现读取配置文件获取pom中的变量

#### 1.配置文件加载顺序

| 配置文件                                                     | 加载顺序 |
| ------------------------------------------------------------ | -------- |
| bootstrap优于application，properties优于yml ，相同配置application不能覆盖bootstrap |          |
| bootstrap.properties                                         | 1        |
| bootstrap.yml                                                | 2        |
| application.properties                                       | 3        |
| application.yml                                              | 4        |

> 1.相同目录优先级
>
> bootstrap.properties/yml >>bootstrap-{profile}.properties/yml>>application.properties/yml>>application-{profile}.properties/yml
>
> 2.不同目录优先级
>
> 在不指定要被加载文件时，默认的加载顺序：由里向外加载，所以最外层的最后被加载，会覆盖里层的属性
>
> 3.后加载的配置覆盖先加载的配置（针对同级不同配置，比如都是applicaiton文件），如果有bootstrap的相关配置已经加载，则使用bootstrap中配置，不会使用application中的配置
>
> 4.适用场景
>
> bootstrap用于应用程序上下文的引导阶段，可以理解成系统级别的参数配置，这些参数一般是不会变动的。
>
> application用于定义应用级别的，搭配 spring-cloud-config 使用 application.yml 里面定义的文件可以实现实时更新，动态切换
>
> 如果bootstrap就能满足，就不用application



#### 2.实现读取配置文件获取pom中的变量(测试只有application中有效，bootstrap中不生效)

------

pom.xml

```xml
<groupId>com.***</groupId>
<artifactId>***-***</artifactId>
<version>8.50-M3-1.2</version>
```

application.yml

```yml
app:
  version: @version@
```

application.properties

```properties
app.version = @version@
```

banner.txt中直接获取

```tex
--version ${app.version}
```

