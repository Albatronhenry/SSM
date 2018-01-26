### [Struts1,2，Servlet，Spring MVC单例多例 线程安全](http://blog.csdn.net/zly9923218/article/details/51125881)

* Struts 1

--------

单例，线程不安全，在请求的时候被第一次初始化

action中的service对象为何不会出现数据存储的错误，大体意思就是每一个用户发出一次请求后就有一个独立的线程与之绑定，且有一个对应的servlet实例，你在其之上做的操作只属于当前servlet实例，不会受其他servlet实例的影响，反之也不会影响其他线程的servlet实例。

* Struts 2

-------------

默认多例，可以设置成单例，线程安全，但是一次不可能很多请求同时过来，那样的话tomcat直接崩溃了。

struts 2的Action中包含数据，例如你在页面填写的数据就会包含在Action的成员变量里面。如果Action是单实例的话，这些数据在多线程的环境下就会相互影响，例如造成别人填写的数据被你看到了。所以Struts2的Action是多例模式的。

* Spring MVC

-------------

默认单例，可以用@Scope(“prototype”)配置成多例，单例的话线程不安全，但是spring mvc不是用action的类属性获取参数，所以没关系。

spring的单例确实存在线程安全的问题。但是spring是如何避免的呢，答案是他用了threadlocal这个类。


* Servlet

-------------

怎样理解Servlet的单实例多线程

servlet单例多线程

单实例，多线程，线程安全，但是操作数据库需要加锁

servlet中的init方法只有在启动（例如web容器启动，要看loadOnStartup的设置）的时候调用，也就是只初始化一次，这就是单实例。

servlet在处理请求的时候 调用的是service方法，这个方法可以处理多个客户端的请求。

具体访问时：

JSP 在web容器中”翻译成servlet”由容器执行，web 容器本身就是提供的多线程,A,B,C 3个访问,建立3个独立的线程组,然后运行一个servlet。依次执行。

1、servlet首先不是现成线程的。

2、Servlet体系结构是建立在Java多线程机制之上的，它的生命周期是由Web容器负责的。

Servlet容器会自动使用线程池等技术来支持系统的运行

3、设定jsp：<%@ page isThreadSafe=”false”%>来实现单线程。

当你需要保证数据一致性的时候，必须自己处理线程安全问题时可以考虑单线程。

---------

### [Struts2的Action是单例还是多例 / SpringMVC的controller默认是单例还是多例？ ](http://blog.csdn.net/chengyuqiang/article/details/78776767)

* 1、默认单例

SpringMVC默认是单例的。与Struts2不同，SpringMVC没有默认处理方法，也就是说SpringMVC是基于方法的开发，都是用形参接收值，一个方法结束参数就销毁了，多线程访问都会有一块内存空间产生，里面的参数也是不会共用的。由于SpringMVC默认使用了单例，所以Controller类中不适合定义属性，只要controller中不定义属性，那么单例完全是安全的。单例模式可以提高SpringMVC性能，不需要每次相应请求都创建一个对象。

此外，Spring的Ioc容器管理的bean默认是单实例的。

* 2、多例

在特殊情况，需要在Controller类定义属性时，单例肯定会出现竞争访问，需要在类上面加上注解@Scope(“prototype”)改为多例的模式。

* 3、Struts2

与SpringMVC不同，Struts2是基于类的属性进行发的，定义属性可以整个类通用。所以Struts2的Action是多实例的并非单例，也就是每次请求产生一个Action的对象。Action类中往往包含了数据属性，例如在页面填写的form表单的字段，Action中有对应的的属性来绑定页面form表单字段。显然如果Action是单实例的话，那么多线程的环境下就会相互影响，例如造成别人填写的数据被你看到了。

但是什么有人说Struts2的Action 默认是单例的？而且还可以进行配置呢？

因为在和Spring一起使用的时候，Action交给Spring进行管理，默认的就是单例，所以才会有人说Struts2默认是单例的。

所以在Spring整合Struts2开发时，如果需要用使用Struts2多例，就在spring的action bean配置的时候设置scope=”prototype”。 
