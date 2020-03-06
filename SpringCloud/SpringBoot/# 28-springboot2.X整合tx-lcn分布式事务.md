# 28-springboot2.X整合tx-lcn分布式事务

###### 1.pom.xml依赖

```xml
		<!-- tx-lcn分布式事务 begin -->
		<codingapi.txlcn.version>5.0.2-SNAPSHOT</codingapi.txlcn.version>

		<dependency>
			<groupId>com.codingapi.txlcn</groupId>
            <!--<artifactId>txlcn-tc</artifactId>下面为公司封装，底层为开源组件-->
			<artifactId>txlcn-tc-crux</artifactId>
			<version>${codingapi.txlcn.version}</version>
			<exclusions>
				<exclusion>
					<groupId>com.google.guava</groupId>
					<artifactId>guava</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.codingapi.txlcn</groupId>
			<artifactId>txlcn-txmsg-netty-crux</artifactId>
			<version>${codingapi.txlcn.version}</version>
			<exclusions>
				<exclusion>
					<groupId>com.google.guava</groupId>
					<artifactId>guava</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.codingapi.txlcn</groupId>
			<artifactId>txlcn-logger-crux</artifactId>
			<version>${codingapi.txlcn.version}</version>
			<exclusions>
				<exclusion>
					<groupId>com.google.guava</groupId>
					<artifactId>guava</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- 在tm中心记录事务控制日志 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.47</version>
		</dependency>
		<!-- tx-lcn分布式事务 end -->
```

###### 2.yml文件配置

```yml
## lcn config
tx-lcn:
  client:
    #tx-manager地址及端口
    manager-address: 127.0.0.1:8070
  logger:
    enabled: true
    only-error: false
    driver-class-name: com.mysql.jdbc.Driver
    #tx-cli连接的数据库
    jdbc-url: jdbc:mysql://127.0.0.1:3306/tx-manager?characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: root111
  rpc:
    reconnect-count: 300
    reconnect-delay: 6000
  springcloud:
    loadbalance:
      enabled: true
```

###### 3.启动类配置

```java
//开启tx-lcn服务发现
@EnableDistributedTransaction
public class DfZjsbApplication {
	public static void main(String[] args) {
		SpringApplication.run(DfZjsbApplication.class, args);
	}

}
```

###### 4.代码块添加注解

```java
@RestController
@RequestMapping("/wf")
public class WfController {
	//分布式事务注解，此处为demo，实际作用于被调用服务service业务实现层比较合适
	@LcnTransaction(propagation = DTXPropagation.REQUIRED)
	@PostMapping("/demo")
	public String Demo() {
		return "jk-zjsb demo test";
	}
}
```



###### TX-manager服务端配置省略，参考如下

> ###### 1.github地址 https://github.com/codingapi/tx-lcn
>
> ###### 2.官网地址 https://www.codingapi.com/ 
