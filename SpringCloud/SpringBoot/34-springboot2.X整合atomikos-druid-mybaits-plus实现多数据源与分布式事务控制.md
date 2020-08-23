## springboot2.X整合atomikos-druid-mybaits-plus实现多数据源与分布式事务控制

背景：因项目需要，切换多数据源对数据进行处理，同时需要保证事务一致性

思路：

> ​		1.通过mybatis包扫描不同扫描路径，实现注册不同数据源
>
> ​		2.将不同数据源注册到sqlSessionFactory
>
> ​		3.将不同数据源注册到事务管理器
>
> ​		4.方法调用测试

------

## 1.pom核心依赖

```xml
    <properties>
        <java.version>1.8</java.version>
        <mybatis-plus>3.2.0</mybatis-plus>
        <druid>1.1.22</druid>
        <mysql>8.0.11</mysql>
    </properties>

    <dependencies>

        <!-- atomikos jta -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jta-atomikos</artifactId>
        </dependency>

        <!-- spring web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- mybatis-plus -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus}</version>
        </dependency>

        <!-- druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid}</version>
        </dependency>

        <!-- oracle-->
        <dependency>
            <groupId>com.oracle.database.jdbc</groupId>
            <artifactId>ojdbc8</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

```

## 2.application.yml配置

```xml
spring:
  aop:
    proxy-target-class: true
    auto: true
  datasource:
    # 配置XA支持
    type: com.alibaba.druid.pool.xa.DruidXADataSource
    druid:
      db1:
        url: jdbc:mysql://localhost:3306/albat?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8
        username: root
        password: root111
        driver-class-name: com.mysql.cj.jdbc.Driver
        initialSize: 5
        minIdle: 5
        maxActive: 20
      db2:
        url: jdbc:oracle:thin:@192.168.112.163:8888/gktest
        username: jdz20
        password: 1
        driver-class-name: oracle.jdbc.OracleDriver
        initialSize: 5
        minIdle: 5
        maxActive: 20
  jta:
    atomikos:
      properties:
        enable-logging: false            #设置不开启，不然会报错，解决参考  https://blog.csdn.net/hzh_shine/article/details/88857627
    transaction-manager-id: txManager    #默认取计算机的IP地址 需保证生产环境值唯一
```

## 3.配置类

### 3.1 MybatisPlusCoreConfig 核心配置

```java
package com.multi.datasource.config;

import com.atomikos.icatch.jta.UserTransactionImp;
import com.atomikos.icatch.jta.UserTransactionManager;
import com.atomikos.jdbc.AtomikosDataSourceBean;
import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.transaction.jta.JtaTransactionManager;

import javax.sql.DataSource;
import javax.transaction.UserTransaction;
import java.util.Properties;

/**
 * MybatisPlus相关配置文件
 * @author linzyc
 */
@Configuration
@EnableTransactionManagement
public class MybatisPlusCoreConfig {

    // druid 分布式事务XA数据源
    private static final String xdscn = "com.alibaba.druid.pool.xa.DruidXADataSource";

    @Bean("db1")
    public DataSource db1(Environment env){
        return getDataSource(env, "spring.datasource.druid.db1.", "db1", xdscn);
    }

    @Bean("db2")
    public DataSource db2(Environment env){
        return getDataSource(env, "spring.datasource.druid.db2.", "db2", xdscn);
    }

    /**
     * 注入事物管理器
     * @return
     */
    @Bean(name = "atomikos")
    public JtaTransactionManager regTransactionManager () {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        UserTransaction userTransaction = new UserTransactionImp();
        return new JtaTransactionManager(userTransaction, userTransactionManager);
    }

    /**
     * 构建jdbc连接
     * @param env
     * @param prefix
     * @return
     */
    private Properties build(Environment env, String prefix) {
        Properties prop = new Properties();
        prop.put("url", env.getProperty(prefix + "url"));
        prop.put("username", env.getProperty(prefix + "username"));
        prop.put("password", env.getProperty(prefix + "password"));
        prop.put("driverClassName", env.getProperty(prefix + "driver-class-name", ""));
        return prop;
    }

    /**
     * 获取数据源
     * @param env
     * @param prefix
     * @param urn
     * @param s
     * @return
     */
    private DataSource getDataSource(Environment env, String prefix, String urn, String s) {
        AtomikosDataSourceBean ds = new AtomikosDataSourceBean();
        Properties prop = build(env, prefix);
        ds.setXaDataSourceClassName(s);
        ds.setUniqueResourceName(urn);
        ds.setPoolSize(5);
        ds.setXaProperties(prop);
        return ds;
    }

    /**
     * 封装好的分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor(){
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        paginationInterceptor.setLimit(-1L);
        return paginationInterceptor;
    }

}
```

### 3.2 MybatisPlusDb1Config 配置

```java
package com.multi.datasource.config;

import com.baomidou.mybatisplus.core.MybatisConfiguration;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;

import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.type.JdbcType;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

/**
 * MybatisPlus 数据源1相关配置文件
 * @author linzyc
 */
@Configuration
@MapperScan(basePackages = "com.multi.datasource.mapper.db1", markerInterface = BaseMapper.class, sqlSessionFactoryRef = "sqlSessionFactorydb1")
public class MybatisPlusDb1Config {

    @Bean("sqlSessionFactorydb1")
    public SqlSessionFactory sqlSessionFactoryDb1(@Qualifier("db1") DataSource db1) throws Exception {
        MybatisSqlSessionFactoryBean mybatisSqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
        mybatisSqlSessionFactoryBean.setDataSource(db1);

        MybatisConfiguration mybatisConfiguration = new MybatisConfiguration();
        mybatisConfiguration.setJdbcTypeForNull(JdbcType.NULL);
        // 不设置驼峰
        mybatisConfiguration.setMapUnderscoreToCamelCase(false);
        mybatisConfiguration.setCacheEnabled(false);

        mybatisSqlSessionFactoryBean.setConfiguration(mybatisConfiguration);

        return mybatisSqlSessionFactoryBean.getObject();
    }
}
```

### 3.3 MybatisPlusDb2Config 配置

```java
package com.multi.datasource.config;

import com.baomidou.mybatisplus.core.MybatisConfiguration;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.type.JdbcType;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;
/**
 * MybatisPlus相关配置文件
 * @author linzyc
 */
@Configuration
@MapperScan(basePackages = "com.multi.datasource.mapper.db2", markerInterface = BaseMapper.class, sqlSessionFactoryRef = "sqlSessionFactorydb2")
public class MybatisPlusDb2Config {

    @Bean("sqlSessionFactorydb2")
    @Primary
    public SqlSessionFactory sqlSessionFactoryDb2(@Qualifier("db2") DataSource db2) throws Exception {
        MybatisSqlSessionFactoryBean mybatisSqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
        mybatisSqlSessionFactoryBean.setDataSource(db2);
        MybatisConfiguration mybatisConfiguration = new MybatisConfiguration();
        mybatisConfiguration.setJdbcTypeForNull(JdbcType.NULL);
        mybatisConfiguration.setMapUnderscoreToCamelCase(false);
        mybatisConfiguration.setCacheEnabled(false);
        mybatisSqlSessionFactoryBean.setConfiguration(mybatisConfiguration);
        return mybatisSqlSessionFactoryBean.getObject();
    }
}
```

## 4. 各相关代码及调用类

### 4.1 MultiDsController

```java
package com.multi.datasource.controller;

import com.multi.datasource.domain.dto.UserDeptEntity;
import com.multi.datasource.domain.dto.UserDeptOrclEntity;
import com.multi.datasource.mapper.db1.UserDeptMapper;
import com.multi.datasource.mapper.db2.UserDeptOrclMapper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;
@RestController()
@RequestMapping("/test")
@Slf4j
public class MultiDsController {
    @Autowired
    private UserDeptMapper userDeptMapper;
    @Autowired
    private UserDeptOrclMapper userDeptOrclMapper;

    @GetMapping("/dosth")
    // 这里的事务管理器名称 atomikos 对应上面配置注册事务管理器名称
    @Transactional(transactionManager = "atomikos", propagation = Propagation.REQUIRED, rollbackFor = { java.lang.RuntimeException.class })
    public void doSomeThing(){
        UserDeptOrclEntity userDeptOrclEntity = new UserDeptOrclEntity();
        userDeptOrclEntity.setDeptName("db2~2~");
        userDeptOrclEntity.setDeptNo("22");
        userDeptOrclMapper.insert(userDeptOrclEntity);
        log.info("running db22");
        UserDeptEntity userDeptEntity = new UserDeptEntity();
        userDeptEntity.setDeptName("db1-1-");
        userDeptEntity.setDeptNo("11");
        //int t = 1/0; 测试是否回滚事务
        userDeptMapper.insert(userDeptEntity);
        log.info("running db11");
    }
}
```

### 4.2 UserDeptEntity && UserDeptOrclEntity

```java
package com.multi.datasource.domain.dto;

import com.baomidou.mybatisplus.annotation.TableName;
import javax.persistence.Column;
import lombok.Data;

@Data
@TableName("userdept")
public class UserDeptEntity {
    @Column(name = "deptname")
    private String deptName;
    @Column(name = "deptno")
    private String deptNo;
}

--------------------------------------------------------
    
package com.multi.datasource.domain.dto;

import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import javax.persistence.Column;

@Data
@TableName("USERDEPTORCL")
public class UserDeptOrclEntity {
    @Column(name = "DEPTNAME")
    private String deptName;
    @Column(name = "DEPTNO")
    private String deptNo;
}
```

### 4.3 UserDeptMapper && UserDeptOrclMapper

注意：这两个接口放在不同的目录下，这样才能保证包扫描分多数据源处理

```java
package com.multi.datasource.mapper.db1;// 这里是db1

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.multi.datasource.domain.dto.UserDeptEntity;

public interface  UserDeptMapper extends BaseMapper<UserDeptEntity> {
}

---------------------------------------------------------------------
    
package com.multi.datasource.mapper.db2;// 这里是db2

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.multi.datasource.domain.dto.UserDeptEntity;
import com.multi.datasource.domain.dto.UserDeptOrclEntity;

public interface UserDeptOrclMapper extends BaseMapper<UserDeptOrclEntity> {
}
```

## 5.启动会看到调用了两次mybatis-plus，然后数据有写入不同数据库

5.1 浏览器输入 http://localhost:8080/test/dosth 控制台调用输出

```cmd
2020-08-23 17:07:28.816  INFO 5768 --- [           main] c.m.datasource.DatasourceApplication     : No active profile set, falling back to default profiles: default
2020-08-23 17:07:30.360  INFO 5768 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-08-23 17:07:30.372  INFO 5768 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-08-23 17:07:30.372  INFO 5768 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.37]
2020-08-23 17:07:30.463  INFO 5768 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-08-23 17:07:30.463  INFO 5768 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1482 ms
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.2.0 
2020-08-23 17:07:30.729  WARN 5768 --- [           main] c.b.m.core.metadata.TableInfoHelper      : Warn: Could not find @TableId in Class: com.multi.datasource.domain.dto.UserDeptEntity.
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.2.0 
2020-08-23 17:07:30.893  WARN 5768 --- [           main] c.b.m.core.metadata.TableInfoHelper      : Warn: Could not find @TableId in Class: com.multi.datasource.domain.dto.UserDeptOrclEntity.
2020-08-23 17:07:31.147  INFO 5768 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-08-23 17:07:31.417  INFO 5768 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-08-23 17:07:31.431  INFO 5768 --- [           main] c.m.datasource.DatasourceApplication     : Started DatasourceApplication in 3.428 seconds (JVM running for 4.732)
2020-08-23 17:07:37.676  INFO 5768 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-08-23 17:07:37.676  INFO 5768 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-08-23 17:07:37.683  INFO 5768 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 7 ms
2020-08-23 17:07:37.724  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : Loaded jar:file:/E:/officesw/maven3.6_local_repository/com/atomikos/transactions/4.0.6/transactions-4.0.6.jar!/transactions-defaults.properties
2020-08-23 17:07:37.728  WARN 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : Thanks for using Atomikos! Evaluate http://www.atomikos.com/Main/ExtremeTransactions for advanced features and professional support
or register at http://www.atomikos.com/Main/RegisterYourDownload to disable this message and receive FREE tips & advice
Thanks for using Atomikos! Evaluate http://www.atomikos.com/Main/ExtremeTransactions for advanced features and professional support
or register at http://www.atomikos.com/Main/RegisterYourDownload to disable this message and receive FREE tips & advice
2020-08-23 17:07:37.733  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.default_max_wait_time_on_shutdown = 9223372036854775807
2020-08-23 17:07:37.733  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.allow_subtransactions = true
2020-08-23 17:07:37.733  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.recovery_delay = 10000
2020-08-23 17:07:37.733  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.automatic_resource_registration = true
2020-08-23 17:07:37.733  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.oltp_max_retries = 5
2020-08-23 17:07:37.733  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.client_demarcation = false
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.threaded_2pc = false
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.serial_jta_transactions = true
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.log_base_dir = ./
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.rmi_export_class = none
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.max_actives = 50
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.checkpoint_interval = 500
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.enable_logging = true
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.log_base_name = tmlog
2020-08-23 17:07:37.734  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.max_timeout = 300000
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.trust_client_tm = false
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: java.naming.factory.initial = com.sun.jndi.rmi.registry.RegistryContextFactory
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.tm_unique_name = 192.168.157.1.tm
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.forget_orphaned_log_entries_delay = 86400000
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.oltp_retry_interval = 10000
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: java.naming.provider.url = rmi://localhost:1099
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.force_shutdown_on_vm_exit = false
2020-08-23 17:07:37.735  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : USING: com.atomikos.icatch.default_jta_timeout = 10000
2020-08-23 17:07:37.736  INFO 5768 --- [nio-8080-exec-1] c.a.icatch.provider.imp.AssemblerImp     : Using default (local) logging and recovery...
2020-08-23 17:07:37.930  INFO 5768 --- [nio-8080-exec-1] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
2020-08-23 17:07:38.745  INFO 5768 --- [nio-8080-exec-1] c.a.d.xa.XATransactionalResource         : db2: refreshed XAResource
2020-08-23 17:07:39.558  INFO 5768 --- [nio-8080-exec-1] c.m.d.controller.MultiDsController       : running db22
2020-08-23 17:07:39.582  INFO 5768 --- [nio-8080-exec-1] com.alibaba.druid.pool.DruidDataSource   : {dataSource-2} inited
Sun Aug 23 17:07:39 CST 2020 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
2020-08-23 17:07:39.724  INFO 5768 --- [nio-8080-exec-1] c.a.d.xa.XATransactionalResource         : db1: refreshed XAResource
Sun Aug 23 17:07:39 CST 2020 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Sun Aug 23 17:07:39 CST 2020 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Sun Aug 23 17:07:39 CST 2020 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Sun Aug 23 17:07:39 CST 2020 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
Sun Aug 23 17:07:39 CST 2020 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
2020-08-23 17:07:39.787  INFO 5768 --- [nio-8080-exec-1] c.m.d.controller.MultiDsController       : running db11

```

5.2 事务管理日志

```json
{"id":"192.168.157.1.tm159817365776300001","wasCommitted":true,"participants":[{"uri":"192.168.157.1.tm1","state":"COMMITTING","expires":1598173669709,"resourceName":"db2"},{"uri":"192.168.157.1.tm2","state":"COMMITTING","expires":1598173669709,"resourceName":"db1"}]}
{"id":"192.168.157.1.tm159817365776300001","wasCommitted":true,"participants":[{"uri":"192.168.157.1.tm1","state":"TERMINATED","expires":1598173669718,"resourceName":"db2"},{"uri":"192.168.157.1.tm2","state":"TERMINATED","expires":1598173669718,"resourceName":"db1"}]}

```

5.3 [参考地址](https://www.cnblogs.com/zhaojiatao/p/8407276.html) ||[github](https://github.com/zhaojiatao/springboot-zjt-chapter10-springboot-atomikos-mysql-mybatis-druid/)