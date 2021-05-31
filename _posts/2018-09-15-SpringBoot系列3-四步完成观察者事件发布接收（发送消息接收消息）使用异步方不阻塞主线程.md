---
layout: post
title: SpringBoot系列3-四步完成观察者事件发布接收（发送消息接收消息）使用异步方不阻塞主线程
tags:
- spring-boot


categories: spring-boot
description:  SpringBoot系列3-四步完成观察者事件发布接收（发送消息接收消息）使用异步方不阻塞主线程
---
 SpringBoot系列3-四步完成观察者事件发布接收（发送消息接收消息）使用异步方不阻塞主线程
<!-- more -->

#### 首先Application开启异步

```java
@SpringBootApplication
@EnableAsync
public class DemoApplication {
 
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
#### 定义事件体（消息）
```java
@Setter
@Getter
@ToString
public class ContentEvent extends ApplicationEvent {
 
    private String content;
 
    /**
     * Create a new ApplicationEvent.
     * @param source the object on which the event initially occurred (never {@code null})
     */
    public ContentEvent(Object source) {
        super(source);
    }
    public ContentEvent(Object source, String content){
        super(source);
        this.content = content;
    }
}
```
#### 定义事件监听者（接收者）

```java
@Component
@Slf4j
public class ContentListener{
 
    @Async
    @EventListener
    public void handler(ContentEvent event){
        log.info("收到消息"+event.getContent());
    }
}
```
#### 发布事件（发送消息）
```java
@Controller
public class ContentController {
    @Autowired
    private ApplicationContext applicationContext;
 
    @GetMapping("/event/{content}")
    public void sendEvent(@PathVariable String content){
        applicationContext.publishEvent(new ContentEvent(this,content));
    }
}
```
 
作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">猿份哥</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="yuanfenge.lskyf.com" target="_blank">yuanfenge.lskyf.com</a> <br>
版权所有，欢迎保留原文链接进行转载：)
关注我们的公众号了解更多<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>