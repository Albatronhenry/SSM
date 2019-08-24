### springboot实现定时任务调度cron

基于项目需求，需要实现定时任务调度，通过保存修改数据实现不重启程序控制定时任务调度。

#### 核心代码如下：
```java
package com.fap.sync.client.controller;

import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ScheduledFuture;

import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.data.domain.Page;
import org.springframework.scheduling.Trigger;
import org.springframework.scheduling.TriggerContext;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.support.CronTrigger;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import com.fap.sync.client.common.entity.Result;
import com.fap.sync.client.common.entity.ResultEnum;
import com.fap.sync.client.common.utils.ContextUtils;
import com.fap.sync.client.entity.JobsEntity;
import com.fap.sync.client.entity.JobsParamsUnionEntity;
import com.fap.sync.client.service.IJobsService;

/**
 * 定时任务执行业务 20190824
 * @author linzy
 */
@EnableScheduling //*关键 开启定时任务
@RestController   //*关键 或使用@component，确保包扫描到
@RequestMapping("/jobs")
public class JobsTaskCornPoolController {
  //*关键 注入线程池时间调度处理
	@Autowired
	private ThreadPoolTaskScheduler threadPoolTaskScheduler;
  
	//*关键 全局定时任务缓存队列，以便控制定时任务停/启而不影响其他定时任务
	private Map<String, ScheduledFuture<?>> futureMap = new ConcurrentHashMap();
	
	@Autowired
	private IJobsService iJobsService;
	@Autowired
	private ContextUtils contextUtils;
	
	//间隔时间表达式
	private String cornexpress = "";
	//*关键 全局间隔时间缓存
	Map cornmap = new HashMap();
	
	/**
	 **关键 注入线程池任务调度器
	 */
	@Bean
	public ThreadPoolTaskScheduler trPoolTaskScheduler() {
		return new ThreadPoolTaskScheduler();
	}
	
	
	/**
	 * 启动
	 */
	@GetMapping("/begin")
	public Result<String> startJob(@RequestParam String j_id) {
		String user_code = contextUtils.getContextInfo().getUsercode();
		String set_year = contextUtils.getContextInfo().getSet_year();
		String rg_code = contextUtils.getContextInfo().getRg_code();
		//更新j_job的j_start=1，启动；
		iJobsService.updateStatueByJid("1", user_code, j_id, set_year, rg_code);
		 String state = execute(j_id);
		 Map bmap = new HashMap();
		 if(StringUtils.isNotEmpty(state)) {
			 bmap.put("successmsg", state);
			 return Result.success(bmap);
		 }else {
			 bmap.put("errormsg", "自动任务执行启动失败！");
			 return Result.error(ResultEnum.BAD_REQUEST,bmap);
		 }
	}
	
	/**
	 * 停止
	 */
	@GetMapping("/stop")
	public Result<String> endJob(@RequestParam String j_id) {
		if(StringUtils.isNotEmpty(j_id) && futureMap.get(j_id) != null) {
			futureMap.get(j_id).cancel(true);
			futureMap.remove(j_id);
			cornmap.remove(j_id);
		}
		String user_code = contextUtils.getContextInfo().getUsercode();
		String set_year = contextUtils.getContextInfo().getSet_year();
		String rg_code = contextUtils.getContextInfo().getRg_code();
		//更新j_job的j_start=0，不启动；
		iJobsService.updateStatueByJid("0", user_code, j_id, set_year, rg_code);
		Map smap = new HashMap();
		smap.put("successmsg", "自动任务停止成功！");
		return Result.success(smap);
	}
	
	/**
	 * 执行
	 */
	@GetMapping("/execute")
	public Result<String> changeJob(@RequestParam String j_id) {
		endJob(j_id);		
		Result<String> start = startJob(j_id);
		Map map = new HashMap();
		if( start.getCode()==200){
			map.put("successmsg", "自动任务执行成功！");
			return Result.success(map);
		}else {
			 map.put("errormsg", "自动任务执行失败！");
			 return Result.error(ResultEnum.BAD_REQUEST,map);
		}
	}
	
	
	/**
	 **关键核心  执行自动任务
	 * @return
	 */
	public String execute(String j_id) {
		String user_code = contextUtils.getContextInfo().getUsercode();
		String set_year = contextUtils.getContextInfo().getSet_year();
		String rg_code = contextUtils.getContextInfo().getRg_code();		
		//*关键 顺序：3.创建任务
		Runnable task = new Runnable() {			
			@Override
			public void run() {
				//*关键 顺序：3.1执行核心业务处理
				iJobsService.excuteCoreBusiness(j_id,user_code,set_year,rg_code);
			}
		};
		//*关键 顺序：2.调用触发器，通过cron表达式控制自动任务时间
		Trigger trigger = new Trigger() {			
			@Override
			public Date nextExecutionTime(TriggerContext triggerContext) {							
				if(StringUtils.isNotEmpty((String) cornmap.get(j_id))) {
					cornexpress = (String) cornmap.get(j_id);
				}else {
					cornexpress = getJtime(user_code,j_id,set_year,rg_code);
          //cron队列中存入当前任务时间表达式
					cornmap.put(j_id, cornexpress);
				}
				CronTrigger cronTrigger = new CronTrigger(cornexpress);
				return cronTrigger.nextExecutionTime(triggerContext);
			}
		};
    //*关键 顺序：1.先实现时间调度，之后调用触发器，创建任务
		ScheduledFuture<?> future = threadPoolTaskScheduler.schedule(task, trigger);
		Map msgMap = new HashMap();
    //*关键 顺序：4.成功启动定时任务，将当前自动任务存入全局任务队列中
		if(future != null) {
			futureMap.put(j_id, future);
			return "自动任务启动成功，执行中...";
		}else {
			return "";
		}
	}
	
	/**
	 * 数据库获取间隔时间
	 * @param j_create
	 * @return
	 */
	private String getJtime(String j_create,String j_id,String set_year, String rg_code) {
		return iJobsService.getJtime(j_create,j_id,set_year,rg_code);
	}
}



```
