### SpringBoot2.X 整合quartz

1. #### pom.xml依赖

```xml
        <!--quartz-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-quartz</artifactId>
        </dependency>
```

2. #### 添加配置类 QuartzConfig

```java
package com.***.abu.ffii.common.config;

import com.***.abu.ffii.common.constants.RequestConstants;
import com.***.abu.ffii.service.cron.CronTask;
import com.***.abu.ffii.service.fiscal.setting.ISettingService;
import org.quartz.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author linzyc
 * @desc 定时任务配置
 * @date 2021-04-17
 */
@Configuration
public class QuartzConfig {

    @Autowired
    private ISettingService iSettingService;

    @Bean
    public JobDetail uploadTaskDetail() {
        // CronTask.class  RequestConstants.CRON_TASK = "CronTask" 配置对应具体实现类
        return JobBuilder.newJob(CronTask.class).withIdentity(RequestConstants.CRON_TASK).storeDurably().build();
    }

    @Bean
    public Trigger uploadTaskTrigger() {
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(iSettingService.getSettingParam().getCron());
        return TriggerBuilder.newTrigger().forJob(uploadTaskDetail())
                .withIdentity(RequestConstants.CRON_TASK).withSchedule(scheduleBuilder).build();
    }
}

```



3. #### 实现类 CronTask

```java
package com.***.abu.ffii.service.cron;

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
import com.***.abu.ffii.common.constants.PayStatueConstants;
import com.***.abu.ffii.common.constants.RequestConstants;
import com.***.abu.ffii.entity.ar.ArBillSettlement;
import com.***.abu.ffii.entity.ffii.pay.FfiiPayStatus;
import com.***.abu.ffii.entity.ffii.user.FfiiMockUser;
import com.***.abu.ffii.mapper.FfiiMockUserMapper;
import com.***.abu.ffii.service.fiscal.interf.IArBillSettlement;
import com.***.abu.ffii.service.fiscal.interf.IFiscalInterface;
import com.***.abu.ffii.service.fiscal.pay.IFiscalPayRequestService;
import com.***.abu.ffii.service.fiscal.setting.ISettingService;
import lombok.SneakyThrows;
import lombok.extern.slf4j.Slf4j;

import org.quartz.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.CollectionUtils;

import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

/**
 * @author linzyc
 * @desc 定时任务-业务实现
 * @date 2021-03-18
 */
@Slf4j
@Component
public class CronTask extends QuartzJobBean {
    
    
	/**这里需要注入*/
    @Autowired
    private Scheduler scheduler;

    @Autowired
    private ISettingService iSettingService;

    @Autowired
    private IFiscalInterface iFiscalPayRequest;

    @Autowired
    private FfiiMockUserMapper ffiiMockUserMapper;

    @Autowired
    private IArBillSettlement arBillSettlementMapper;

    @Autowired
    private IFiscalPayRequestService iFiscalPayRequestService;

    @Override
    @SneakyThrows
    protected void executeInternal(org.quartz.JobExecutionContext jobExecutionContext) {
         doBusiness(ffiiMockUser);
    }

    private void doBusiness(FfiiMockUser ffiiMockUser) {
        log.info("定时任务");
    }

    /**
     * 更新定时任务
     * RequestConstants.CRON_TASK = "CronTask" 配置对应具体实现类
     * iSettingService.getSettingParam().getCron() 动态设置定时任务时间片
     */
    @SneakyThrows
    public void updateScheduleJob()  {
        TriggerKey triggerKey = TriggerKey.triggerKey(RequestConstants.CRON_TASK);
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule(iSettingService.getSettingParam().getCron());
        CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
        trigger = trigger.getTriggerBuilder().withIdentity(triggerKey).withSchedule(scheduleBuilder).build();
        scheduler.rescheduleJob(triggerKey, trigger);
    }
}

```

4. #### 启动项目，自动任务执行

![image-20210417204114865](https://raw.githubusercontent.com/Albatronhenry/UploadFile/master/pic/image-20210417204114865.png)

5. [参考](https://cloud.tencent.com/developer/article/1640190)