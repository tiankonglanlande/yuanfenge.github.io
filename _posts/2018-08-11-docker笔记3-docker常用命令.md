---
layout: post
title: docker笔记3-docker常用命令
tags:
- docker 


categories: docker
description:  docker笔记3-docker常用命令
---
项目中docker比较常用的一些命令
<!-- more -->

###### 查看容器（ps） ######
###### 列出当前所有正在运行的container ######
```
docker ps 
```
###### 列出所有的container ######
```
docker ps -a 
```
###### 列出最近一次启动的container ###### 
```
docker ps -l
```
###### 查看本地主机上的镜像镜像 ######
```
docker images
```
###### 从dockerhub中搜索镜像 ######
```
docker search redis
```
###### 拉取镜像 ######
冒号后面指定镜像版本号，不指定则拉取最新的镜像版本
```
docker pull mysql:5.7.22
docker pull mysql
```
###### 停止、启动、杀死一个容器 ###### 
```
docker stop Name/ID 
docker start Name/ID 
docker kill Name/ID
```
###### 删除容器 ######
```
docker rm -v Name/ID
```

###### 删除镜像 ###### 
```
docker rmi Name/ID
//冒号后面跟的是镜像版本号
docker rmi Name:1.0  
```
###### docker build镜像 ###### 
```
//docker build –tag=镜像名称 .  注意前面的点别丢
docker build –tag=elastic-custom .
```
###### 启动elasticsearch ###### 
```
docker run -p 9200:9200 -p 9300:9300 -ti -v /usr/share/elasticsearch/data elastic-custom
```
###### docker 进入容器 ###### 
```
//docker exec -it 容器编号 /bin/sh 
docker exec -it c23a7729ea83 /bin/sh    
```
###### 将镜像保存为tar文件 ######
```
docker save test:1.0 -o test.tar
```
######  将镜像tar文夹加载到docker ####
```
docker load < test.tar
```
<br/>
<br/>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">猿份哥</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="yuanfenge.lskyf.com" target="_blank">yuanfenge.lskyf.com</a> <br>
版权所有，欢迎保留原文链接进行转载：)
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>
