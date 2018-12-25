---
layout: post
title: spring boot设置favicon，favicon不生效，不成功，不起作用
tags:
- java
- spring-boot
- favicon

categories: spring-boot
description:  spring boot设置favicon，favicon不生效，不成功，不起作用，为什么？
---
springboot显示的是一片叶子，我们如何使用自己的favicon呢？

我试过网上文章设置都不成功，经过实践应该是这样设置：

#### 1.将favicon.icon放到resources目录下  例如：/public,/static等等 #### 

####  2.完成上面的步骤还不能显示，还需在你的页面的head标签添加代码 #### 
```
<head>
   <meta charset="UTF-8">
   <title>登录</title>
   <link rel="shortcut icon" th:href="@{/favicon.ico}"/>
   <link rel="bookmark" th:href="@{/favicon.ico}"/>
</head>
```
#### 3.注意我使用的thymeleaf所以是以上代码片段如果你不是请这样添加 ####

```
<head>
   <meta charset="UTF-8">
   <title>登录</title>
   <link rel="shortcut icon" href="/favicon.ico"/>
   <link rel="bookmark" href="/favicon.ico"/>
</head> 
```
<br/>
<br/>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">猿份哥</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="http://www.lskyf.xyz" target="_blank">http://www.lskyf.xyz</a> <br>
版权所有，欢迎保留原文链接进行转载：) 
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>