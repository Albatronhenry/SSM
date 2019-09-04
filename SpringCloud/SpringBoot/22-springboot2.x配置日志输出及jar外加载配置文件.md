### springboot2.x配置日志输出及jar外加载配置文件

因项目中需要，配置日志输出及加载jar包外配置文件，方便维护

[配置日志输出，根据日期](https://github.com/Albatronhenry/SSM/blob/master/SpringCloud/SpringBoot/22-springboot2.x%E9%85%8D%E7%BD%AE%E6%97%A5%E5%BF%97%E8%BE%93%E5%87%BA%E5%8F%8Ajar%E5%A4%96%E5%8A%A0%E8%BD%BD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.md)-->*.yml配置如下
```yml

# 日志输出
logging:
  file: ./logger/fap_sync_client.log #./表示根目录，如果需要具体路径，指定具体路即可
  pattern: #配置日志格式 %d：日期 , %msg：日志信息 ，%n换行
    file:  "%d{yyyy-MM-dd HH:mm:ss} %-5level %logger(36) - %msg%n"  #设置log文件路径 设置日志文件名称cell.log

```

[jar外加载配置文件](https://github.com/Albatronhenry/SSM/blob/master/SpringCloud/SpringBoot/22-springboot2.x%E9%85%8D%E7%BD%AE%E6%97%A5%E5%BF%97%E8%BE%93%E5%87%BA%E5%8F%8Ajar%E5%A4%96%E5%8A%A0%E8%BD%BD%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.md)-->*.bat/cmd脚本配置 java –jar -Dspring.config.location=xxx/xxx/xxxx.properties xxxx.jar
```cmd
@echo off

title fap_sync_client启动 linzy

@echo -------------------------fap_sync_client开始启动-------------------------
set BASE_DIR=%cd%
java -jar -Dspring.config.location=%BASE_DIR%/setting/application-dev.yml %BASE_DIR%/fap_sync_client-0.0.1-SNAPSHOT.jar

```
