一、Quartz功能概述
Quartz是一个开源的任务调度系统，它能用来调度很多任务的执行。

运行环境
Quartz 能嵌入在其他应用程序里运行。
Quartz 能在一个应用服务器里被实例化(或servlet容器), 并且参与XA事务
Quartz能独立运行（通过JVM）,或者通过RMI
Quartz能被集群实例化
任务调度
当一个指定给任务的触发器发生时,任务就被调度执行.触发器能被创建为:

一天的某个时间(精确到毫秒级)
一周的某些天
一个月的某些天
一年的某些天
不在一个Calendar列出的某些天 (例如工作节假日)
在一个指定的次数重复
重复到一个指定的时间/日期
无限重复
在一个间隔内重复
能够给任务指定名称和组名.触发器也能够指定名称和组名,这样可以很好的在调度器里组织起来.一个加入到调度器里的任务可以被多个触发器注册。在J2EE环境里，任务能作为一个分布式（XA）事务的一部分来执行。

任务执行
任务能够是任何实现Job接口的Java类。
任务类能够被Quartz实例化,或者被你的应用框架。
当一个触发器触发时，调度器会通知实例化了JobListener 和TriggerListener 接口的0个或者多个Java对象(监听器可以是简单的Java对象, EJBs, 或JMS发布者等). 在任务执行后，这些监听器也会被通知。
当任务完成时，他们会返回一个JobCompletionCode ，这个代码告诉调度器任务执行成功或者失败.这个代码 也会指示调度器做一些动作-例如立即再次执行任务。
任务持久化
Quartz的设计包含JobStore接口，这个接口能被实现来为任务的存储提供不同的机制。
应用JDBCJobStore, 所有被配置成“稳定”的任务和触发器能通过JDBC存储在关系数据库里。
应用RAMJobStore, 所有任务和触发器能被存储在RAM里因此不必在程序重起之间保存-一个好处就是不必使用数据库。
事务
使用JobStoreCMT（JDBCJobStore的子类），Quartz 能参与JTA事务。
Quartz 能管理JTA事务(开始和提交)在执行任务之间，这样，任务做的事就可以发生在JTA事务里。
集群
Fail-over.
Load balancing.
监听器和插件
通过实现一个或多个监听接口，应用程序能捕捉调度事件来监控或控制任务/触发器的行为。
插件机制可以给Quartz增加功能，例如保持任务执行的历史记录，或从一个定义好的文件里加载任务和触发器。
Quartz 装配了很多插件和监听器。
二、Quartz体系结构
Quartz对任务调度的领域问题进行了高度的抽象，提出了调度器、任务和触发器这3个核心的概念，并在org.quartz通过接口和类对重要的这些核心概念进行描述：

●Job

       是一个接口，只有一个方法voidexecute(JobExecutionContext context)，开发者实现该接口定义运行任务，JobExecutionContext类提供了调度上下文的各种信息。Job运行时的信息保存在 JobDataMap实例中；

●JobDetail

        Quartz在每次执行Job时，都重新创建一个Job实例，所以它不直接接受一个Job的实例，相反它接收一个Job实现 类，以便运行时通过newInstance()的反射机制实例化Job。因此需要通过一个类来描述Job的实现类及其它相关的静态信息，如Job名字、描 述、关联监听器等信息，JobDetail承担了这一角色。

通过该类的构造函数可以更具体地了解它的功用：JobDetail(java.lang.String name, java.lang.String group,java.lang.Class jobClass)，该构造函数要求指定Job的实现类，以及任务在Scheduler中的组名和Job名称；

●Trigger

       是一个类，描述触发Job执行的时间触发规则。主要有SimpleTrigger和CronTrigger这两个子类。当仅需触 发一次或者以固定时间间隔周期执行，SimpleTrigger是最适合的选择；而CronTrigger则可以通过Cron表达式定义出各种复杂时间规 则的调度方案：如每早晨9:00执行，周一、周三、周五下午5:00执行等；

●Calendar

       org.quartz.Calendar和java.util.Calendar不同，它是一些日历特定时间点的集合（可以简 单地将org.quartz.Calendar看作java.util.Calendar的集合——java.util.Calendar代表一个日历时间点，无特殊说明后面的Calendar即指org.quartz.Calendar）。一个Trigger可以和多个Calendar关联，以便排除或 包含某些时间点。

假设，我们安排每周星期一早上10:00执行任务，但是如果碰到法定的节日，任务则不执行，这时就需要在Trigger触发机制的基础上使用 Calendar进行定点排除。针对不同时间段类型，Quartz在org.quartz.impl.calendar包下提供了若干个Calendar 的实现类，如AnnualCalendar、MonthlyCalendar、WeeklyCalendar分别针对每年、每月和每周进行定义；

●Scheduler

        代表一个Quartz的独立运行容器，Trigger和JobDetail可以注册到Scheduler中，两者在 Scheduler中拥有各自的组及名称，组及名称是Scheduler查找定位容器中某一对象的依据，Trigger的组及名称必须唯一，JobDetail的组和名称也必须唯一（但可以和Trigger的组和名称相同，因为它们是不同类型的）。Scheduler定义了多个接口方法， 允许外部通过组及名称访问和控制容器中Trigger和JobDetail。

        Scheduler可以将Trigger绑定到某一JobDetail中，这样当Trigger触发时，对应的Job就被执行。一个Job可以对应 多个Trigger，但一个Trigger只能对应一个Job。可以通过SchedulerFactory创建一个Scheduler实例。 Scheduler拥有一个SchedulerContext，它类似于ServletContext，保存着Scheduler上下文信息，Job和 Trigger都可以访问SchedulerContext内的信息。SchedulerContext内部通过一个Map，以键值对的方式维护这些上下文数据，SchedulerContext为保存和获取数据提供了多个put()和getXxx()的方法。可以通过Scheduler# getContext()获取对应的SchedulerContext实例；

三、入门实例

新建一个Java maven project.

导入相应的坐标

```xml

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.yonyou</groupId>
  <artifactId>testquartz</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
  	<dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-core</artifactId>
  		<version>3.2.0.RELEASE</version>
  	</dependency>
  	<dependency>
  		<groupId>org.quartz-scheduler</groupId>
  		<artifactId>quartz</artifactId>
  		<version>2.2.1</version>
  	</dependency>
  	<dependency>
  		<groupId>commons-logging</groupId>
  		<artifactId>commons-logging</artifactId>
  		<version>1.2</version>
  	</dependency>
  	 <dependency>
  		<groupId>org.springframework</groupId>
  		<artifactId>spring-context</artifactId>
  		<version>3.2.0.RELEASE</version>
  	</dependency>
  </dependencies>
</project>

```

执行业务逻辑的Job

```java

package com.yonyou;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

public class MyJob implements Job{
	@Override
	public void execute(JobExecutionContext arg0) throws JobExecutionException {
		  System.out.println("Albatron quzrtz test update timer:  "+  
			        new SimpleDateFormat("yyyy-MM-dd HH:mm:ss ").format(new Date()));  
		
	}
}

```
-------------

```java

package com.yonyou;

import org.quartz.JobBuilder;
import org.quartz.JobDetail;
import org.quartz.Scheduler;
import org.quartz.SchedulerException;
import org.quartz.SimpleScheduleBuilder;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;
import org.quartz.impl.StdSchedulerFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest{
	  public static void main(String[] args) {  
		  MyTest quartzTest=new MyTest();  
	        quartzTest.startSchedule();  
	  
	    }  
	  
	    public void startSchedule() {  
	        try {  
	            // 1、创建一个JobDetail实例，指定Quartz  
	            JobDetail jobDetail = JobBuilder.newJob(MyJob.class)  
	            // 任务执行类  
	            .withIdentity("job1_1", "jGroup1")  
	            // 任务名，任务组  
	            .build();     
	            //2、创建Trigger  
	            SimpleScheduleBuilder builder=SimpleScheduleBuilder.simpleSchedule()  
	            //设置间隔执行时间  
	            .withIntervalInSeconds(2)  
	            //设置执行次数  
	            .repeatForever();  
	            Trigger trigger=TriggerBuilder.newTrigger().withIdentity(  
	                    "trigger1_1","tGroup1").startNow().withSchedule(builder).build();  
	            //3、创建Scheduler  
	            Scheduler scheduler=StdSchedulerFactory.getDefaultScheduler();  
	            scheduler.start();        
	            //4、调度执行  
	            scheduler.scheduleJob(jobDetail, trigger);                
	            try {  
	                Thread.sleep(10000);  
	            } catch (InterruptedException e) {  
	                e.printStackTrace();  
	            }  
	   
	            scheduler.shutdown();  
	              
	        } catch (SchedulerException e) {  
	            e.printStackTrace();  
	        }  
	    }  
}

```

运行结果如下：

!(结果)[https://github.com/Albatronhenry/UploadFile/blob/master/pic/quartz_result.png]
