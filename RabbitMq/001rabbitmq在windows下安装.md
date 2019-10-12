### [rabbitmq在windows下安装](https://www.cnblogs.com/dqcer/p/9358811.html)

* 1.点击otp_win64_20.1.exe安装Erlang,因为rabbitmq依赖
* 2.点击rabbitmq-server-3.6.14.exe安装
* 3.安装后，cmd查看是否安装好
```cmd
"E:\officesw\rabbitmq\rabbitmq_server-3.6.14\sbin\rabbitmq-plugins.bat" enable rabbitmq_management
会有相应输出，说明安装成功
```
* 4.重启Rabbitmq服务，即可使用
```cmd
net stop RabbitMQ && net start RabbitMQ
```
