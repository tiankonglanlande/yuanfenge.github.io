---
layout: post
title: SpringBoot系列6-SpringBoot中使用servlet
tags:
- spring-boot
- servlet

categories: spring-boot
description:  在SpringBoot中使用servlet
---
 介绍在SpringBoot中如何使用servlet
<!-- more -->
#### pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency> 
```

#### 定义类MyServlet继承HttpServlet

@WebServlet标记servlet，urlPatterns配置映射路径

重写doGet方法

```java
@WebServlet(urlPatterns = "/my/servlet")
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
        resp.getWriter().print("hello this is MyServlet!");
    }
}
```

#### 配置启动类

@ServletComponentScan(basePackages = "com.tiankonglanlande.cn.springboot.servlet.servlet")这个注解作用是
配置要扫描servlet的位置，springboot会自动将包下的servlet注入

```java

@SpringBootApplication
@ServletComponentScan(basePackages = "com.tiankonglanlande.cn.springboot.servlet.servlet")
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}
}

```

#### 异步的servlet

配置@WebServlet的属性asyncSupported = true即可

```java
/**
 * @author 猿份哥
 */
@WebServlet(urlPatterns = "/my/ayncservlet",asyncSupported = true)
public class MyAyncServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);

        AsyncContext asyncContext = req.getAsyncContext();
        asyncContext.start(()->{
            try {
                resp.getWriter().print("hello this is MyAyncServlet!");
                asyncContext.complete();
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }
}
 
```
#### 测试
在浏览器访问：http://localhost:8080/my/servlet

输出：hello this is MyServlet!

在浏览器访问：http://localhost:8080/my/ayncservlet

输出：hello this is MyAyncServlet!

#### 可能遇到的问题
1.HTTP method GET is not supported by this URL
解决方法：doGet方法中去掉super.doGet()方法调用

2.java.lang.IllegalStateException: It is illegal to call this method if the current request is not in asynchronous mode (i.e. isAsyncStarted() returns false)
原因：（1）将req.startAsync()错写成req.getAsyncContext();
      （2）asyncContext.complete()需要在任务完成后调用

[源码下载链接](https://github.com/tiankonglanlande/springboot)

作者：猿份哥，版权所有，欢迎保留原文链接进行转载：)
关注我们的公众号了解更多<br>
