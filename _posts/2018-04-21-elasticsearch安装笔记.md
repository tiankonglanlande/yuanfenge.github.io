---
layout: post
title: elasticsearch安装笔记
tags:
- elasticsearch
- elasticsearch安装笔记
categories: elasticsearch
description:  自己安装elasticsearch的实践过程
---
## elasticsearch安装笔记 ##
前言：自己安装elasticsearch的实践过程
<!-- more -->

Elasticseatch 安装笔记 
## 1.	Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root. ##
不能使用root用户启动需要创建一个用户test给其授权elasticsearch的文件给test
#### (1)root添加用户 ####
添加用户 test： 
adduser test 
#### (2)修改test密码：  ####
passwd test
#### (3) 给用户添加文件夹权限 chown [-R] < 用户名或组>< 文件或目录> ####
用root用户执行 ： chown -R test elasticsearch
#### (4)切换到test用户 ####
su – test
#### (5)启动elasticsearch切换到elasticsearch/bin目录下./elasticsearch ####
此种方式当窗口关闭elasticsearch就停止了
#### (6) 后台启动 ####
切换到elasticsearch/bin目录下./elasticsearch –d
#### (7)服务的方式启动（有时间再介绍）	 ####
## 2.	设置外网访问 ##
vim /config/elasticsearch.yml做以下修改
network.host: 0.0.0.0
http.port: 9200   #端口号
network.publish_host: 120.78.91.62   #此处填写你的外网ip即可

注意：如果是阿里云还需设置实例添加安全组允许9200外网访问
## 3.	vim 修改文件出现错误“E45: 'readonly' option is set (add ! to override) ##
如果是root权限，可以使用:wq! 强行保存退出

作者：天空蓝蓝的  www.lskyf.com   www.lskyf.xyz  
版权所有，欢迎保留原文链接进行转载：)