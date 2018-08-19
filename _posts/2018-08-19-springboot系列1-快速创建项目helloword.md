---
layout: post
title: SpringBoot系列1-helloword
tags:
- spring-boot


categories: spring-boot
description:  SpringBoot系列1-helloword
---
使用springboot简单轻松创建helloword
<!-- more -->
	SpringBoot系列1-helloword

##	为什么要使用springboot呢？ ##
###### （1）	影响力大：sping团队的力作，Spring Boot致力于在蓬勃发展的快速应用开发领域成为领导者 ######
###### （2）	简单：使编码，配置，部署，监控变简单 ######
###### （3）	快速：能够简单快速构建好一个项目而不使用太复杂的配置 ######
###### （4）自动化：根据你的配置自动导入相关依赖 ######


##	实现一个简单的helloword  ##

环境：jdk1.8，maven3.1+（非必须）
开发工具：IDE(Eclipse、IntelliJ 或者其它的)
###### （1）	新建一个maven项目，可以是web工程也可以是java基本工程 ######

###### （2）	配置pom.xml ######
```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
```
###### （3）新建HelloController ######
```
    @RestController
    public class HelloController {
        @RequestMapping("/say/{content}")
        public String helloword(@PathVariable("content") String content){
            return content;
        }
    }
```
###### （4）SpringbootApplication.java ######
```
 @SpringBootApplication
 public class SpringbootApplication {

 	public static void main(String[] args) {
 		SpringApplication.run(SpringbootApplication.class, args);
 	}
 }

```

###### （5）运行测试在浏览器访问 ######
http://localhost:8080/say/helloword
可以看到浏览器得到返回结果【helloword】

源码地址：<a href="https://github.com/tiankonglanlande/springboot" target="_blank">https://github.com/tiankonglanlande/springboot</a> <br>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">天空蓝蓝的</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="http://www.lskyf.xyz" target="_blank">http://www.lskyf.xyz</a> <br>
版权所有，欢迎保留原文链接进行转载：)
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>