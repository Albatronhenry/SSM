[springboot_https_ico](http://blog.720ui.com/2017/springboot_05_server_tomcat_https/)
-------------------------
### html页面title标签添加图片配置
* 在Springboot-jpa-thymeleaf项目   /src/main/resources/static/ico 下添加 favicon.ico
* 在html中配置  
```html
<head>
    <meta charset="UTF-8"/>
    <title>Y-@</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}"></link>
    
   <link rel="shortcut icon" th:href="@{/ico/favicon.ico}" type="image/x-icon"/>
   
   
</head>
```

### 配置https
* 1.生成证书,使用jdk自带的keytools工具    

[参考1](https://www.sslshopper.com/article-how-to-create-a-self-signed-certificate-using-java-keytool.html) | [参考2](http://kzpkzp.blog.163.com/blog/static/16869581820105294490538/)

` keytool -genkey -alias springboot -storetype PKCS12 -keyalg RSA -keysize 1024 -keystore keystore.p12 -validity 365 `

命令说明
-----------
```
-alias        指定证书的别名
-keyalg        指定密钥算法名称, 此处使用 RSA
-storetype    指定证书类型
-keysize        指定私钥位数
-validity    指定有效期, 单位为天. 此处指定有效期为 365 天
-keystore    指定密钥库位置
```

配置参考下图:

![keytools_https](https://github.com/Albatronhenry/UploadFile/blob/master/pic/krytools_https.png)

配置好后,在相应目录下找到生成的文件   ` keystore.p12 `,把它复制到项目的classpath路径下

* 2.在 application.properties 中配置 HTTPS 支持
```properties
server.port=8443
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=123456
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=springboot
```
* 3.启动与测试,controller等类参照
[10-spring_jpa_thymeleaf](https://github.com/Albatronhenry/SSM/blob/master/SpringCloud/SpringBoot/10-spring_jpa_thymeleaf.md)
配置,完成后登录
![ico_https](https://github.com/Albatronhenry/UploadFile/blob/master/pic/ico_https.png)
