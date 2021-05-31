---
layout: post
title: docker笔记4-Dockerfile详解
tags:
- docker 


categories: docker
description:  docker笔记4-Dockerfile详解
---
项目中docker比较常用的一些命令
<!-- more -->


#### FROM 
指定基础镜像<br/>
eg：<br/>
FROM java <br/>


#### MAINTAINER(已过时)
指定镜像的创建者<br/>
eg:<br/>
MAINTAINER nihao@sina.com

#### RUN
可以运行任何被基础image支持的命令，<br/>
两种方式<br/>
RUN <br/>
RUN ["executable","param1","param2"]


#### 设置容器启动时执行的操作
三种格式：
CMD [“executable”, “param1”, “param2”] 使用 exec 执行，推荐方式。<br/>
CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用。<br/>
CMD [“param1”, “param2”] 提供给 ENTRYPOINT 的默认参数。<br/>

eg:<br/>
CMD ["java","-jar","app.jar"]


#### ADD
格式为：<br/>
将指定文件复制到容器中，指定的文件可以是Dockerfile所在目录的相对路径中（文件或目录）也可以是一个URL；还可以是一个tar文件会将tar文件自动解压 <br/>
ADD target/*.jar /app.jar <br/>


#### COPY
复制本地主机的 (为 Dockerfile 所在目录的相对路径，文件或目录) 为容器中的 。目标路径不存在时，会自动创建。当使用本地目录为源目录时，推荐使用 COPY。<br/>
<br/>
eg:<br/>
COPY target/*.jar /app.jar 

#### ENTRYPOINT
两种方式 <br/>
ENTRYPOINT [“executable”, “param1”, “param2”] <br/>
ENTRYPOINT command param1 param2 (shell 中执行) <br/>

1.配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。 <br/>
2.每个Dockerfile中只能有一个ENTRYPOINT，当指定多个时，只有最后一个生效 <br/>


#### EXPOSE
暴露端口号 <br/>
eg:<br/>
EXPOSE 8080 <br/>

#### ENV
构建指令,在image中设置一个环境变量 <br/>
设置了后,后续的RUN命令都可以使用,container启动后,可以通过docker inspect查看这个环境变量,也可以通过在docker run –env key=value时设置或修改环境变量 <br/>
eg:<br/>
安装的JAVA程序,设置环境变量  <br/>
ENV JAVA_HOME /data/programs/jdk <br/>
ENV PATH $PATH:$JAVA_HOME/bin <br/>
ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 


#### VOLUME
创建一个可以从本地或其他容器挂载的挂载点，一般用来存放数据库和需要保存的数据等
<br/>
eg:<br/>
VALUME["/data"] <br/>
VOLUME ["/data1","/data2"] <br/>


#### USER
eg:<br/>
USER daemon <br/>
指定运行容器时的用户名或UID，后续的RUN也会使用指定用户。当服务不需要管理员权限时可以通过该命令运行指定用户。并且可以在之前创建所需要的用户，<br/>
例如：RUN groupadd -f test && useradd -r test -g test


#### WORKDIR
切换工作目录<br/>
eg：<br/>
WORKDIR /a/b/c



#### LABEL
格式 <br/>
LABEL key=value  <br/>
LABEL 指令为镜像添加标签。一个 LABEL 就是一个键值对。<br/>
eg:<br/>
LABEL version="1.0"<br/>
LABEL content="描述"<br/>
每条 LABEL 指令都会生成一个新的层。所以最好是把添加的多个 LABEL 合并为一条命令：<br/>
LABEL multi.label1="value1" multi.label2="value2" other="value3"<br/>
也可以使用下面方式<br/>
LABEL multi.label1="value1"\
multi.label2="value1"\
other.label3="value3"

<br/>
<br/>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">猿份哥</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="yuanfenge.lskyf.com" target="_blank">yuanfenge.lskyf.com</a> <br>
版权所有，欢迎保留原文链接进行转载：)
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>
