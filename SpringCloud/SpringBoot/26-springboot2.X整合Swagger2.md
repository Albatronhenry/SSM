### [springboot2.X整合Swagger2](https://blog.csdn.net/moyanxiaoq/article/details/84136670)
项目中，Eureka注册发布相应，springboot整合swagger2能够很好的发布API文档
--------------------------------------------------------
* 1.pom添加相应依赖（因公司自带依赖包包含，故不用引用
```xml
<!--swagger2依赖-->
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.8.0</version>
</dependency>

<!--swagger2-ui依赖-->
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.8.0</version>
</dependency>

<!--lombok依赖-->
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.16.20</version>
</dependency>

```
* 2.yml配置文件配置（可以不用配置
```yml
spring:
  #  配置开发模式( dev test prod
  profiles:
    active: dev 

```
* 3.后端代码
 * 3.1创建配置文件
 ---------------
 ```java
 package com.yonyougov.report.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * Swagger2文档配置
 * @author linzyc
 */
@Configuration
//下面两个注解可不配置
@EnableSwagger2
@Profile({"dev"})//同yml配置文件里
public class Swagger2Config {

	@Bean
	public Docket config() {
		return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo()).select()
    //扫描对应的controller类作为API暴露接口
				.apis(RequestHandlerSelectors.basePackage("com.yonyougov.report.controller")) 
				.paths(PathSelectors.any()).build();
	}

	private ApiInfo apiInfo() {
		Contact contact = new Contact("linzyc", "", "linzyc@yonyou.com");
		return new ApiInfoBuilder()
				.title("报表文档")
				.contact(contact)
				.version("1.0.0-SNAPSHOT")
				.build();
	}
}

 ```
  * 3.2对应controller类编辑相应swagger注解
  -------------------------------------
  ```java
  package com.yonyougov.report.controller;

import java.util.List;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.yonyougov.report.domain.BaseUser;
import com.yonyougov.report.service.impl.IBaseUserService;
import com.yonyougov.report.util.feign.DfReportFeignTest;
import com.yonyougov.report.util.feign.FbpBasedataFeignN;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiOperation;

@RestController
@RequestMapping("/api/user")
@Api(value="用户测试pi",tags= {"用户操作测试接口"},description="用户基本信息操作 @author linzyc")
public class BaseUserController {
	@Autowired
	private IBaseUserService iBaseUserService;
	@Autowired
	private DfReportFeignTest dfReportFeignTest;
	@Autowired
	private FbpBasedataFeignN FbpBasedataFeignNew;
	
	/**
	 * 获取用户信息
	 */
	@ApiOperation(value="获取用户信息",notes="测试获取用户信息")
	@GetMapping("/list")
	public Object selectUser() {
		Page page = new Page(1,5);
		page = iBaseUserService.selectUser(page, "henry");
		return page;
	}
	/**
	 * 新增用户
	 */
	@ApiOperation(value="新增用户",notes="测试创建用户")
	@ApiImplicitParam(name="baseUser",value="用户实体",required = true, paramType = "body", dataType = "BaseUser")
	@PostMapping("/add")
	public Object addUser(@RequestBody BaseUser baseUser) {
		baseUser.setName("政务");
		Long count = iBaseUserService.addUser(baseUser);
		return count;
	}
	
	/**
	 * 测试接口回调
	 */
	@ApiOperation(value="测试接口回调",notes="测试接口回调")
	@GetMapping("/test")
	public Page test() {
		List<Map<String, Object>>  ls = FbpBasedataFeignNew.getElement();
		Page page = new Page(1,5);
		page =  dfReportFeignTest.selectUser();
		return page;
	}
}

  ```
  
* 4.启动服务访问应用地址，即可成功访问http://localhost:8080/df-report/swagger-ui.html
