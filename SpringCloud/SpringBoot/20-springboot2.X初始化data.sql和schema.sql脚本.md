### springboot2.X初始化data.sql和schema.sql脚本
[参考1](https://www.cnblogs.com/klyjb/p/11002778.html)|[参考2](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-database-initialization.html)
----------
schema-client.sql来源于mysql,如需转成oracle脚本执行[mysql脚本数据导出成oracle执行脚本](https://github.com/Albatronhenry/SQL/issues/7)

keypoint：
```yml
spring:
  datasource:
# mysql  
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF8&useSSL=false&serverTimezone=UTC
    username: root
    password: root111
    platform: mysql
# 重点关注下面两行分别指定创建文件所在位置及2.x比配 
    schema: classpath:schema-client.sql
    initialization-mode: always
# 使用了spring加载脚本就不要使用hibanate-jpa，因此设置为none    
  jpa:
    hibernate:
      ddl-auto: none  
```


