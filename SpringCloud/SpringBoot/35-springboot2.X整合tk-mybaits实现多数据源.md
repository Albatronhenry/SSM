## springboot2.X整合tk-mybaits实现多数据源

思路：

> 				1.通过mybatis包扫描不同扫描路径，实现注册不同数据源
> 			
> 				2.将不同数据源注册到sqlSessionFactory
> 			
> 				3.将不同数据源注册到事务管理器
> 			
> 				4.方法调用测试

------

## 1.pom核心依赖

```xml
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>

        <!-- spring web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- mybatis-plus -->
		<dependency>
			<groupId>tk.mybatis</groupId>
			<artifactId>mapper-spring-boot-starter</artifactId>
			<version>${tk.mybatis.version}</version>
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
    db1:
      jdbc-url: jdbc:mysql://localhost:3306/albat?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8
      username: root
      password: root111
      driver-class-name: com.mysql.cj.jdbc.Driver
      initialSize: 5
      minIdle: 5
      maxActive: 20
    db2:
      jdbc-url: jdbc:oracle:thin:@192.168.112.163:8888/gktest
      username: jdz20
      password: 1
      driver-class-name: oracle.jdbc.OracleDriver
    hikari:
      minimum-idle: 10
      maximum-pool-size: 100
```

## 3.配置类

### 3.1 DataSource1Config 核心配置

```java
package com.multi.datasource.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import tk.mybatis.spring.annotation.MapperScan;

import javax.sql.DataSource;

/**
 * @author linzyc
 * @desc 数据源1配置
 * @date 2021-03-04
 */
@Configuration
@MapperScan(basePackages = "com.multi.datasource.dao.*", sqlSessionTemplateRef  = "ds1SqlSessionTemplate")
public class DataSource1Config {
    @Bean(name = "DataSource1")
    @ConfigurationProperties(prefix = "spring.datasource.db1")
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "ds1SqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("DataSource1") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
//        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/ds1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "ds1TransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(@Qualifier("DataSource1") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "ds1SqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("ds1SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

### 3.2  MybatisPlusDb2Config 配置

```java
package com.multi.datasource.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import tk.mybatis.spring.annotation.MapperScan;

import javax.sql.DataSource;

/**
 * @author linzyc
 * @desc 数据源2配置
 * @date 2021-03-04
 */
@Configuration
@MapperScan(basePackages = "com.multi.datasource.mapper1.*", sqlSessionTemplateRef  = "ds2SqlSessionTemplate")
public class DataSource2Config {
    @Bean(name = "DataSource2")
    @ConfigurationProperties(prefix = "spring.datasource.db2")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "ds2SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("DataSource2") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
//        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/ds2/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "ds2TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("DataSource2") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "ds2SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(@Qualifier("ds2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

## 4. 各相关代码及调用类

### 4.1 MultiDsController

```java
package com.multi.datasource.controller;

import com.multi.datasource.dao.agency.Agency1Mapper;
import com.multi.datasource.domain.entity.agency.Agency1;
import com.multi.datasource.mapper1.agency.Agency2Mapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author linzyc
 * @desc ...
 * @date 2021-03-04
 */
@RestController
@RequestMapping("/multiDs")
public class MultiDsTestController {

    @Autowired
    private Agency1Mapper agency1Mapper;

    @Autowired
    private Agency2Mapper agency2Mapper;

    @GetMapping("/add")
    public String Test(){
        Agency1 agency = new Agency1();
        agency.setAGENCY_ID("123");
        agency.setAGENCY_CODE("101001");
        agency.setAGENCY_NAME("数据同步多数据源");
        agency1Mapper.insert(agency);
        agency2Mapper.insert(agency);
        return "ok";
    }
}

```

### 4.2 Agency1实体类

### 4.3 Agency1Mapper && Agency2Mapper

注意：这两个接口放在不同的目录下，这样才能保证包扫描分多数据源处理

### 5.启动会看到调用了两次mybatis，然后数据有写入不同数据库

5.1 浏览器输入  http://127.0.0.1:7000/df-ywsp-multi/multiDs/add 控制台调用输出