### dubbo&zookeeper相关
-----------
近来研习相关工具，不是很清楚之间的关系，通过网上学习，简单梳理。
dubbo官方定义式分布式中间件，不涉及分布式，可以不使用。那么其与zookeeper关系是怎样的？

                                        ▏dubbo▕
                                         ▁▁▁
                                    ↗ 1    ↑   2↘
                                       ↙3  |
                         provider         --4-→  consumer
                         ▁▁▁▁            |    ▁▁▁▁
                                            |
                      Server                |
                      ▁▁▁                 ↓
                      server A   --→     zookeeper  ←--  Client（Server）
                      server B    ↗     ▁▁▁▁▁       ▁▁▁
                      ...
                     
* 当client（server）调用另一个server时，存在多个server，client（server）不清楚调哪个（不清楚怎么负载均衡），维护很麻烦，
所以zookeeper注册中心能够很好的解决这个问题[zookeeper在dubbo的作用](https://www.cnblogs.com/shiyu404/p/8945542.html)
* zookeeper注册中心宕机，dubbo仍能短时间内仍能正常，因为dubbo获取相关信息后会缓存在本地，不依赖注册中心，除非新增改变等操作
* dubbo的Provider，Consumer启动时都会注册一个zookeeper注册中心，1，2，3，4过程是实现过程[Dubbo与注册中心Zookeeper的交互过程](https://blog.csdn.net/qq_27529917/article/details/80632078)

[Dubbo中zookeeper做注册中心，如果注册中心集群全都挂掉，发布者和订阅者之间还能通信么？](https://blog.csdn.net/zh521zh/article/details/77948208)
----------------
* 【提供者】在【启动】时，向注册中心zk 【注册】自己提供的服务。
* 【消费者】在【启动】时，向注册中心zk 【订阅】自己所需的服务。
  * 消费者在启动时，消费者会从zk拉取注册的生产者的地址接口等数据，缓存在本地。每次调用时，按照本地存储的地址进行调用
  * 消费者本地有一个生产者的列表，他会按照列表继续工作，倒是无法从注册中心去同步最新的服务列表，短期的注册中心挂掉是不要紧的，但一定要尽快修复
  * 挂掉是不要紧的，但前提是你没有增加新的服务，如果你要调用新的服务，则是不能办到的
* 以上体现健壮性
