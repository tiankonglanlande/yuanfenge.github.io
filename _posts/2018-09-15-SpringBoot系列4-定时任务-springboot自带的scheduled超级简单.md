---
layout: post
title: SpringBoot系列4-定时任务-springboot自带的scheduled超级简单
tags:
- spring-boot


categories: spring-boot
description:  SpringBoot系列4-定时任务-springboot自带的scheduled超级简单
---
 SpringBoot系列4-定时任务-springboot自带的scheduled超级简单
<!-- more -->

#### 需求：创建一个每天凌晨0点执行的定时任务

#### 1.创建任务

```java
/**
 * @author 天空蓝蓝的
 */
@Slf4j
@EnableScheduling
@Component
public class MyTask {
    @Async
    @Scheduled(cron = "0 0 0 * * ?")
    public void delEveryDay() throws SchedulerException, InterruptedException {
        log.info("每天凌晨0点开始执行任务");
        //业务代码
 
    }
}

```
@EnableScheduling  启用定时任务，可以添加到Application类上 ，此处添加到MyTask类上。

@Scheduled 添加到方法上 ，表示要执行的方法

@Async 并行执行（异步的），如果想串行执行无需添加

cron表示执行的条件，此处为每天凌晨0点执行

#### 2.Application入口类
```java
@SpringBootApplication
@EnableAsync
public class Start {
    public static void main(String[] args) {
        SpringApplication.run(Start.class, args);
    }
}

```

@EnableAsync 开启并行执行（异步的），如果想串行执行无需开启

#### 3.扩展
此外我们还可以这样实现定时任务
例如我们想5秒执行一次任务，代码如下
``` java
    @Scheduled(fixedDelay = 5000)
    public void towTask(){
        System.out.println("5秒后执行定时任务1");
    }

```
#### 4.项目经验温馨提醒
我们在创建spring的定时任务需要遵循这些规则否则定时任务不会生效

1.类需要使用@Component注解

2.定时任务方法需要注解@Scheduled并且方法不能有返回值和参数


[源码下载链接](https://github.com/tiankonglanlande/springboot)


版权所有，欢迎保留原文链接进行转载：)
关注我们的公众号了解更多<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>