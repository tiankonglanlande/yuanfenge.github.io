---
layout: post
title: docker笔记5-使用docker-compose build image像并启动应用
tags:
- docker 


categories: docker
description:  docker笔记5-使用docker-compose build image像并启动应用
---
使用docker-compose build image像并启动应用
<!-- more -->

使用docker-compose build image像并启动应用
#### 准备材料
website.jar , Dockerfile , docker-compose.yml

Dockerfile
```
FROM java:8-jre-alpine
MAINTAINER test@xx.com
ENV JAVA_OPTS null
WORKDIR /home/portal/website
ADD /home/portal/website/website-2.0.2.jar /home/portal/website/
EXPOSE 8081
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar website-2.0.2.jar" ]
VOLUME /home/portal/website:/home/portal/website

```

docker-compose.yml

```
version: '3'
services:
  website:
    build:
      context: .
      dockerfile: Dockerfile
    image: website:2.0.2
    container_name: website
    restart: always
    networks:
      - nets
    volumes: 
      - /home/portal/website/logs/:/home/portal/website/logs/
    ports:
      - "8081:8081"
    environment:    
      - JAVA_OPTS=-Xmx256M -Dspring.profiles.active=test -Duser.timezone=GMT+08
networks: 
  nets:
    external: false
```

#### 将以上材料放到/home/portal/website目录

#### build镜像启动应用：执行docker-compose up -d

<br/>
<br/>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">天空蓝蓝的</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="http://www.lskyf.xyz" target="_blank">http://www.lskyf.xyz</a> <br>
版权所有，欢迎保留原文链接进行转载：)
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>
