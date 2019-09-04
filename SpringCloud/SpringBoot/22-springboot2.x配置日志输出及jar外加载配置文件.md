### springboot2.x配置日志输出及jar外加载配置文件

因项目中需要，配置日志输出及加载jar包外配置文件，方便维护

[配置日志输出，根据日期]()-->*.yml配置如下
```yml

# 日志输出
logging:
  file: ./logger/fap_sync_client.log #./表示根目录，如果需要具体路径，指定具体路即可
  pattern: #配置日志格式 %d：日期 , %msg：日志信息 ，%n换行
    file:  "%d{yyyy-MM-dd HH:mm:ss} %-5level %logger(36) - %msg%n"  #设置log文件路径 设置日志文件名称cell.log

```

[jar外加载配置文件]()-->*.bat/cmd脚本配置 java –jar -Dspring.config.location=xxx/xxx/xxxx.properties xxxx.jar
```cmd
@echo off

title fap_sync_client启动 linzy

@echo -------------------------fap_sync_client开始启动-------------------------
set BASE_DIR=%cd%
java -jar -Dspring.config.location=%BASE_DIR%/setting/application-dev.yml %BASE_DIR%/fap_sync_client-0.0.1-SNAPSHOT.jar

```
