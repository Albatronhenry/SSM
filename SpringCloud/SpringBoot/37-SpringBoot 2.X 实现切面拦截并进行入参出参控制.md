### SpringBoot 2.X 实现切面拦截并进行入参出参控制

1. pom依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

2. 代码处理

```java
package com.***.abu.ffii;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.***.abu.ffii.common.exception.BaseException;
import com.***.abu.ffii.common.result.Result;
import com.***.abu.ffii.entity.ffii.agency.FfiiAuthAgency;
import com.***.abu.ffii.mapper.FfiiAuthAgencyMapper;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * @author linzyc
 * @desc 统一对单位进行拦截授权校验及区划码处理
 * @date 2021-03-26
 */
@Slf4j
@Aspect
@Component
public class AopInterceptController {

    private static final String ZERO_STR = "000";

    @Autowired
    private FfiiAuthAgencyMapper ffiiAuthAgencyMapper;

    /** 切点 */
    @Pointcut("execution(* com.***.abu.ffii.controller.fiscal.*Controller.*(..))")
    private void point(){

    }

    /** 使用环绕通知，需要对入参进行特殊处理 */
    @Around(value = "point()")
    public Result checkAuthAgencyAndDealRgCode(ProceedingJoinPoint joinPoint) throws Throwable {
        Object[] args = joinPoint.getArgs();
        args[0] = String.valueOf(args[0]).concat(ZERO_STR);
        String rgCode = String.valueOf(args[0]);
        String agencyCode = String.valueOf(args[1]);
        Integer count = ffiiAuthAgencyMapper.selectCount(new QueryWrapper<FfiiAuthAgency>().eq("mof_div_code", rgCode).eq("code", agencyCode));
        if(count < 1){
            throw new BaseException("当前单位未匹配到相关信息，请确认当前单位是否开通...");
        }
        log.info("拦截请求参数并处理后的参数1、参数2：{},{}",args[0],args[1]);
        /** 需要增加代理返回值，不然切面拦截了，导致无返回值现象 */
        return (Result) joinPoint.proceed(args);
    }
}
```

3.   ProceedingJoinPoint JoinPoint 

> JoinPoint 用于 @Before  @After
>
> ProceedingJoinPoint 继承 JoinPoint ，功能更强大，可对入参进行处理，用于@Around，不能用于@Before  @After

4. 参考

> [切面实现参考](https://blog.csdn.net/lpp_dd/article/details/75409589?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)
>
> [返回值问题参考](https://blog.csdn.net/m0_37647376/article/details/103496031)
