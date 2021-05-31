---
layout: post
title: docker笔记1-centos7安装docker
tags:
- linux 
- docker
- centos7


categories: docker
description:  docker笔记1-linux安装docker
---
docker笔记记录，使用的时候查一查。
<!-- more -->

#### 1.卸载老版本的docker及相关依赖 #### 
```
yum remove docker docker-cilent docker-cilent-latest  docker-common  docker-latest   docker-common-logrotate container-selinux docker-selinux docker-engine
```
#### 2.安装yum-utils，它提供了yum-config-manager,可用来管理yum源 #### 
```
yum install -y yum-utils   device-mapper-persistent-data lvm2
```
####  3.添加yum源 ####  
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
####  4.更新yum索引 #### 
```
yum makecache fast
```
#### 5.安装docker-ce #### 
```
yum install -y docker-ce docker-ce-di containerd.io
```
#### 6.启动docker #### 
```
systemctl start docker
```
#### 7.验证是否成功 #### 
```
docker info
docker --version
```
#### 配置阿里云镜像加快速度镜像拉取https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors链接中可以看到。
```
mkdir -p /etc/docker
```

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://1iat0xok.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
#### 配置docker开机启动
```
systemctl enable docker
```
<br/>
<br/>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">猿份哥</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="yuanfenge.lskyf.com" target="_blank">yuanfenge.lskyf.com</a> <br>
版权所有，欢迎保留原文链接进行转载：)
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>