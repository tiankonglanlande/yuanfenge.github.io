---
layout: post
title: docker笔记2-docker安装自定义elasticsearch，ik分词
tags:
- docker 
- elasticsearch
- Dockerfile
- ik分词


categories: docker  Dockerfile  elasticsearch 
description:  docker笔记2-docker安装elasticsearch，ik分词
---
docker笔记记录，记录docker安装自定义elasticsearch，包含Dockerfile,ik分词安装。
<!-- more -->

#### 1.编写elasticsearch.yml #### 
```
cluster.name: "docker-cluster"
network.host: 0.0.0.0
# minimum_master_nodes need to be explicitly set when bound on a public IP
# # set to 1 to allow single node clusters
# # Details: https://github.com/elastic/elasticsearch/pull/17288
# discovery.zen.minimum_master_nodes: 1
# xpack.license.self_generated.type: basic
```
#### 2.编写Dockerfile #### 
```
FROM docker.elastic.co/elasticsearch/elasticsearch:6.2.4
COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
RUN cd /usr/share/elasticsearch && ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.4/elasticsearch-analysis-ik-6.2.4.zip
COPY --chown=elasticsearch:elasticsearch synonyms.txt /usr/share/elasticsearch/config/analysis-ik/
ADD --chown=elasticsearch:elasticsearch mydic /usr/share/elasticsearch/config/analysis-ik/mydic/
```
#### 3.将Dockerfile文件，elasticsearch.yml，mydic文件夹（存放扩展词文件），synonyms.txt文件到系统同一个目录 ####

#### 4.docker build镜像 #### 
```
docker build --tag=elastic-custom .
```
#### 5.启动elasticsearch #### 
```
docker run -p 9200:9200 -p 9300:9300 -ti -v /usr/share/elasticsearch/data elastic-custom
```
####  6.docker 进入容器containerId替换为容器id #### 
```
docker exec -it containerId /bin/sh
```
#### 7.进入容器elastic-custom重置密码 #### 
```
./bin/x-pack/users useradd my_admin -p my_password -r superuser
curl -u my_admin -XPUT 'http://localhost:9200/_xpack/security/user/elastic/_password?pretty' -H 'Content-Type: application/json' -d' 
{ "password": "newpassword" }' 
```

#### 8.验证elastic新密码输入新密码看看是否成功 #### 
```
curl -u elastic 'http://localhost:9200/_xpack/security/_authenticate?pretty'
```

作者：天空蓝蓝的  www.lskyf.com   www.lskyf.xyz  
版权所有，欢迎保留原文链接进行转载：)