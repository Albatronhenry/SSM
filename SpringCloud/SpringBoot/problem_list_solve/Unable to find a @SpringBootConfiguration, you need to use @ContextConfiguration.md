[Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration ](https://blog.csdn.net/leigh_/article/details/69409488)
--------------
异常如下:

```java
java.lang.IllegalStateException: Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=...) with your test
at org.springframework.util.Assert.state(Assert.java:70)
at org.springframework.boot.test.context.SpringBootTestContextBootstrapper.getOrFindConfigurationClasses(SpringBootTestContextBootstrapper.java:173)
```

解决方法:
  测试和主函数启动类所在包名一致就可以了,包名一致
  
 * eg:
    主函数类包名
   * com.yonyou.helloworld
   测试类包名也应为
   * com.yonyou.helloworld
   
