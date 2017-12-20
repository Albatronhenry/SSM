## Spring ApplicationContext cannot be resolved even with jar

* 在搭建SSM项目时，出现如下错误：

![报错](https://github.com/Albatronhenry/UploadFile/blob/master/pic/applicationcontext1.PNG)

开始搭建时是参照别人的项目文档搭建的，他们构建的是普通项目，我构建的是Maven项目，所以这里就留下了一个隐患(pom文件中容易漏加对应jar配置)；
之后通过 

(stackoverflow 网站)
[https://stackoverflow.com/questions/12084265/spring-applicationcontext-cannot-be-resolved-even-with-jar]

找到了解决方案

![解决](https://github.com/Albatronhenry/UploadFile/blob/master/pic/applicationcontext2.PNG)


配置如下：

![配置](https://github.com/Albatronhenry/UploadFile/blob/master/pic/applicationcontext3.PNG)

问题得以顺利解决！

