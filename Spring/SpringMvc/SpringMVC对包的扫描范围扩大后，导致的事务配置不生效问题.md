[SpringMVC对包的扫描范围扩大后，导致的事务配置不生效问题](https://www.cnblogs.com/zhaojz/p/8176052.html)
-----------------
<!--扫描注解包 配置这条便可移除 <context:annotation-config/> --> <context:component-scan base-package="com.yonyou.action" /> 

上述配置的结果是：SpringMVC对Service和Dao的所有package进行了扫描装载。

问题分析：
1、Spring与SpringMVC属于父子容器关系。框架启动时先启动Spring容器，而后启动SpringMVC容器。子容器可以访问父容器中的Bean，而父容器不能访问子容器中的Bean。

2、由于SpringMVC在扫描时扩大了扫描范围，装载了@Service标识的类的实例，从而导致Controller层在注入Service时，实际注入的时子容器中的Service实例。

3、事务被配置在父容器中，Spring父容器在装载Service时会同时应用事务配置，而SpringMVC只是单纯加载Service的实例。

问题解决：
将SpringMVC的包扫描限定在Controller。

  <context:component-scan base-package="com.yonyou.action.controller" /> 
