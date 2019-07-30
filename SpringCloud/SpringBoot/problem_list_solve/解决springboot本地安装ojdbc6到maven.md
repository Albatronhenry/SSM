[解决springboot本地安装ojdbc6到maven](https://blog.csdn.net/u011109627/article/details/79710172)
项目中用oracle数据库，驱动无法引入，后查询发现是不支持，需要本地再安装。
```xml
1.将oracle包导入到maven中命令
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.1.0 -Dpackaging=jar -Dfile=D:\ojdbc6.jar
2.增加配置
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0.1.0</version>
</dependency>

```
