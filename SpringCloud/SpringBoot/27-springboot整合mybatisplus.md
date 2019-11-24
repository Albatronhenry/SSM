### springboot整合mybatisplus

* 1.pom文件增加依赖
```xml
	<!-- mybatis-plus 3.2.0-->
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>X.X.X</version>
		</dependency>
```
* 2.application.yml文件配置
```yml
#mybatis-plus
mybatis-plus:
  #映射文件扫描
  mapper-locations: classpath:/mapper/*.xml
  #包扫描路径
  type-aliases-package: com.yonyougov.report.*.*
  # 该配置请和 typeAliasesPackage 一起使用，如果配置了该属性，则仅仅会扫描路径下以该类作为父类的域对象 。
  type-aliases-super-type: java.lang.Object
  configuration:
     # 这个配置会将执行的sql打印出来，在开发或测试的时候可以用
     log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
     # 驼峰下划线转换
     map-underscore-to-camel-case: true
     # 配置的缓存的全局开关
     cache-enabled: true
     # 延时加载的开关
     lazy-loading-enabled: true
     # 开启的话，延时加载一个属性时会加载该对象全部属性，否则按需加载属性
     multiple-result-sets-enabled: true
     use-generated-keys: true
     default-statement-timeout: 60
     default-fetch-size: 100
```
* 3.创建全局配置类MybatisPlusConfig
```java
package com.***.common.config;

import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

/**
 * MybatisPlus相关配置文件
 * @author linzyc
 */
@Configuration
@EnableTransactionManagement
public class MybatisPlusConfig {
    /**
     * 封装好的分页插件
     */
    @Bean
    public PaginationInterceptor paginationInterceptor(){
    	PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
    	paginationInterceptor.setLimit(-1L);//设置为-1单页条数不限制，解决分页默认限制查询500条问题
        return paginationInterceptor;
    }

}
```
