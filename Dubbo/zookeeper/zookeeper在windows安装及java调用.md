### [zookeeper在windows安装简单使用](https://blog.csdn.net/qiunian144084/article/details/79192819)
----------
* 安装jdk
* 安装Zookeeper. 在官网http://zookeeper.apache.org/ 下载zookeeper.
* 解压zookeeper-3.4.14至F:\entirement-sw\zookeeper\zookeeper-3.4.14\.
* ZooKeeper的安装模式分为三种，分别为：单机模式（stand-alone）、集群模式和集群伪分布模式。ZooKeeper 
单机模式的安装相对比较简单，如果第一次接触ZooKeeper的话，建议安装ZooKeeper单机模式或者集群伪分布模式。

单击模式
------------
* 至F:\entirement-sw\zookeeper\zookeeper-3.4.14\conf 复制 zoo_sample.cfg 并粘贴到当前目录下，命名zoo.cfg.
* 至bin目录下，启动zkServer.cmd,再启动zkClient.cmd
* 启动完成后cmd命令下，netstat-ano查看端口监听服务

伪集群模式
------------
* 伪集群模式在单机模式基础上，复制对应zkServer.cmd，重命名zkServer1.cmd，zkServer2.cmd...
 * 因为zkServer.cmd实际回调zkEnv.cmd，所以复制然后重命名zkEnv1.cmd，zkEnv2.cmd...
 * 同理zoo.cfg也要类似创建
* 启动相应zkServer*.cmd，然后执行zkcli.cmd -server:localhost:2181;zkcli.cmd ;-server:localhost:2182;zkcli.cmd -server:localhost:2183.等等

java连接实现（待补充）
-----------
