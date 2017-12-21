###SSM-BeanFactory和ApplicationContext的区别

  * BeanFactory               -- BeanFactory采取延迟加载，第一次getBean时才会初始化Bean
        * ApplicationContext        -- 在加载applicationContext.xml时候就会创建具体的Bean对象的实例，还提供了一些其他的功能
            * 事件传递
            * Bean自动装配
            * 各种不同应用层的Context实现
            
            
            
1.BeanFactory和ApplicationContext的异同点： 

   相同点：

    两者都是通过xml配置文件加载bean,ApplicationContext和BeanFacotry相比,提供了更多的扩展功能。

   不同点：

    BeanFactory是延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；
    而ApplicationContext则在初始化自身是检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用ApplicationContext

 2.在实际开发中用BeanFactory还是ApplicationContext
		  ApplicationContext包含BeanFactory的所有功能。通常建议比BeanFactory优先，除非有一些限制的场合如字节长度对内存有很大的影（Applet）.然后，绝大多数"典型的"企业应用和系统，ApplicationContext就是你需要使用的。
		  Spring2.0及以上版本，大量使用了link  linkend="beans-factory-extension-bpp">BeanPostProcessor扩展（以便应用代理等功能），
		  如果你选择BeanFactory则无法使用相当多的支持功能，如事务和AOP，这可能会导致混乱，因为配置并没有错误。
		  
 3.一个简单的例子来证明BeanFactory和ApplicationContext主要区别

    搭建工程的环境就不说了，直接上代码。

1.首先创建一个实体类：User


```java
public class User {

     public User(){

      System.out.println("实例化User");

     }

}
```
 

2.在创建一个ApplicationContext.xml文件


```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"

    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation="

        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="cn.test.User"></bean>

</beans>
```
 

3.创建测试类
```java
public class TestHappy {

 



 * BeanFactory的测试



@Test

public void beanFactoryTest(){

BeanFactory beanFactory=new XmlBeanFactory(new ClassPathResource("applicationContext.xml"));

		}
```
 

 



 * ApplicationContext的测试


```java
@Test

public void applicationContextTest(){

ApplicationContext context=new ClassPathXmlApplicationContext("applicationContext.xml");

		}
} 

```

 

BeanFactory测试结果：
#### 什么都没显示

ApplicationContext测试结果：
#### 实例化User
 


 ` 总结：`

	   BeanFactory当需要调用时读取配置信息，生成某个类的实例。如果读入的Bean配置正确，则其他的配置中有错误也不会影响程序的运行。
	   而ApplicationContext 在初始化时就把 xml 的配置信息读入内存，为XML 文件进行检验，如果配置文件没有错误，就创建所有的Bean , 直接为应用程序服务。
	   相对于基本的BeanFactory，ApplicationContext 唯一的不足是占用内存空间。当应用程序配置Bean较多时，程序启动较慢。
