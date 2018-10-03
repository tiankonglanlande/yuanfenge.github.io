---
layout: post
title: SpringBoot系列5-定时任务-springboot整合quartz实现动态定时任务
tags:
- spring-boot
- quartz

categories: spring-boot
description:  SpringBoot系列5-定时任务-springboot整合quartz实现动态定时任务
---
 SpringBoot系列5-定时任务-springboot整合quartz实现动态定时任务
<!-- more -->

#### springboot有自带的定时任务为什么还要使用quartz

> 使用springboot自带的定时任务可以很简单很方便的完成一些简单的定时任务，但是我们想动态的执行我们的定时任务就比较困难了。然而使用quartz却可以很容易的管理我们的定时任务，很容易动态的操作定时任务。下面我们就讲解下如何使用quartz动态实现定时任务！

#### 首先来一张截图看看我们的目录结构
<img src="{{ site.assets }}/images/2018-10-03/1.png"/>

#### pom.xml引入依赖
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--quartz相关依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
<!--lombok-->
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```
#### MyJob实现Job接口,重写execute方法在里面操作我们要执行的业务逻辑。
```java
@Slf4j
public class MyJob implements Job {
    @Autowired
    private MyService myService;

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        log.info("任务开始执行了");
        try {
            executeTask();
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
        log.info("任务执行结束了");
    }

    private void executeTask() throws SchedulerException {
        myService.bizFunction();
    }
}
```
#### 定义QuartzManager管理我们的定时任务方法
```java
**
 * @author 天空蓝蓝的
 */
@Configuration
public class QuartzManager {

    public static final String JOB1="job1";
    public static final String GROUP1="group1";
    /**默认每个星期凌晨一点执行*/
    //public static final String DEFAULT_CRON="0 0 1 ? * L";
    /**默认5秒执行一次*/
    public static final String DEFAULT_CRON="*/5 * * * * ?";

    /**
     * 任务调度
     */
    @Autowired
    private Scheduler scheduler;

    /**
     * 开始执行定时任务
     */
    public void startJob() throws SchedulerException {
        startJobTask(scheduler);
        scheduler.start();
    }

    /**
     * 启动定时任务
     * @param scheduler
     */
    private void startJobTask(Scheduler scheduler) throws SchedulerException {
        JobDetail jobDetail= JobBuilder.newJob(MyJob.class).withIdentity(JOB1,GROUP1).build();
        CronScheduleBuilder cronScheduleBuilder=CronScheduleBuilder.cronSchedule(DEFAULT_CRON);
        CronTrigger cronTrigger=TriggerBuilder.newTrigger().withIdentity(JOB1,GROUP1)
                .withSchedule(cronScheduleBuilder).build();
        scheduler.scheduleJob(jobDetail,cronTrigger);

    }
    /**
     * 获取Job信息
     * @param name
     * @param group
     */
    public String getjobInfo(String name,String group) throws SchedulerException {
        TriggerKey triggerKey=new TriggerKey(name,group);
        CronTrigger cronTrigger= (CronTrigger) scheduler.getTrigger(triggerKey);
        return String.format("time:%s,state:%s",cronTrigger.getCronExpression(),
                scheduler.getTriggerState(triggerKey).name());
    }

    /**
     * 修改任务的执行时间
     * @param name
     * @param group
     * @param cron cron表达式
     * @return
     * @throws SchedulerException
     */
    public boolean modifyJob(String name,String group,String cron) throws SchedulerException{
        Date date=null;
        TriggerKey triggerKey=new TriggerKey(name, group);
        CronTrigger cronTrigger= (CronTrigger) scheduler.getTrigger(triggerKey);
        String oldTime=cronTrigger.getCronExpression();
        if (!oldTime.equalsIgnoreCase(cron)){
            CronScheduleBuilder cronScheduleBuilder=CronScheduleBuilder.cronSchedule(cron);
            CronTrigger trigger=TriggerBuilder.newTrigger().withIdentity(name,group)
                    .withSchedule(cronScheduleBuilder).build();
            date=scheduler.rescheduleJob(triggerKey,trigger);
        }
        return date !=null;
    }

    /**
     * 暂停所有任务
     * @throws SchedulerException
     */
    public void pauseAllJob()throws SchedulerException{
        scheduler.pauseAll();
    }

    /**
     * 暂停某个任务
     * @param name
     * @param group
     * @throws SchedulerException
     */
    public void pauseJob(String name,String group)throws SchedulerException{
        JobKey jobKey=new JobKey(name,group);
        JobDetail jobDetail=scheduler.getJobDetail(jobKey);
        if (jobDetail==null)
            return;
        scheduler.pauseJob(jobKey);
    }

    /**
     * 恢复所有任务
     * @throws SchedulerException
     */
    public void resumeAllJob()throws SchedulerException{
        scheduler.resumeAll();
    }
    /**
     * 恢复某个任务
     */
    public void resumeJob(String name,String group)throws SchedulerException{
        JobKey jobKey=new JobKey(name,group);
        JobDetail jobDetail=scheduler.getJobDetail(jobKey);
        if (jobDetail==null)
            return;
        scheduler.resumeJob(jobKey);
    }

    /**
     * 删除某个任务
     * @param name
     * @param group
     * @throws SchedulerException
     */
    public void deleteJob(String name,String group)throws SchedulerException{
        JobKey jobKey=new JobKey(name, group);
        JobDetail jobDetail=scheduler.getJobDetail(jobKey);
        if (jobDetail==null)
            return;
        scheduler.deleteJob(jobKey);
    }

}

```
#### 定义ApplicationStartQuartzJobListener实现当应用启动,或刷新时，启动我们的定时任务
```java
/**
 * @author 天空蓝蓝的
 */
@Slf4j
@Configuration
public class ApplicationStartQuartzJobListener implements ApplicationListener<ContextRefreshedEvent> {

    @Autowired
    private QuartzManager quartzManager;


    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        try {
            quartzManager.startJob();
            log.info("任务已经启动......");
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
    }
    /**
     * 初始注入scheduler
     */
    @Bean
    public Scheduler scheduler() throws SchedulerException{
        SchedulerFactory schedulerFactoryBean = new StdSchedulerFactory();
        return schedulerFactoryBean.getScheduler();
    }
}

```
#### 定义MyJobFactory创建Quartz实例
```java

/**
 * @author 天空蓝蓝的
 */
@Component
public class MyJobFactory extends AdaptableJobFactory{
    @Autowired
    private AutowireCapableBeanFactory capableBeanFactory;

    @Override
    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
        // 调用父类的方法
        Object jobInstance = super.createJobInstance(bundle);
        // 进行注入
        capableBeanFactory.autowireBean(jobInstance);
        return jobInstance;
    }
}

```
#### Application如何类Start注入MyJobFactory和Scheduler
```java

/**
 * @author 天空蓝蓝的
 */
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@Configuration
public class Start {

	public static void main(String[] args) {
		SpringApplication.run(Start.class, args);
	}

	@Autowired
	private MyJobFactory myJobFactory;

	@Bean
	public SchedulerFactoryBean schedulerFactoryBean() {
		SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
		schedulerFactoryBean.setJobFactory(myJobFactory);
		System.out.println("myJobFactory:"+myJobFactory);
		return schedulerFactoryBean;
	}
	@Bean
	public Scheduler scheduler() {
		return schedulerFactoryBean().getScheduler();
	}
}

```
#### 定义ModifyCronController动态修改定时任务
```java
/**
 * @author 天空蓝蓝的
 */
@RestController
public class ModifyCronController {

    @Autowired
    private QuartzManager quartzManager;
    @GetMapping("modify")
    public String modify() throws SchedulerException {
        /**10秒执行一次*/
        String cron="*/10 * * * * ?";
        quartzManager.pauseJob(QuartzManager.JOB1,QuartzManager.GROUP1);
        quartzManager.modifyJob(QuartzManager.JOB1,QuartzManager.GROUP1,cron);
        return "ok";
    }
}

```
#### 浏览器访问http://localhost:8080/modify我们可以看到将默认的5秒执行一次任务动态变成了10秒执行一次了

[源码下载链接](https://github.com/tiankonglanlande/springboot)


作者：天空蓝蓝的，版权所有，欢迎保留原文链接进行转载：)
关注我们的公众号了解更多<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>