---
layout: post
title: 使用logstash从 mysql 全量同步数据到elasticsearch
tags:
- elasticsearch 
- logstash
- mysql
- 全量同步数据
categories: elasticsearch 
description:   使用logstash从 mysql 全量同步数据到elasticsearch 
---
需求说明：想要实现一个防止重复提交的功能：网上的实例也很多个人觉得很麻烦，要是能简单到只需要添加一个标记到需要防止重复提交的方法上，不管你请求来自移动端还是pc端，而且不需要防止重复的地方我就不添加标记那该多好啊！ 
<!-- more -->

#### 1.安装elasticsearch,
##### 1.1、下载并安装ES的yum公钥 #####
 ```
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
 ```
##### 1.2、配置ES的yum源 #####
 ```
vim /etc/yum.repos.d/elasticsearch.repo
 ```
 输入下面的内容：
 ```
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```


#### 1.3.1、更新yum的缓存 #### 
yum makecache
#### 1.3.2、安装ES ####
yum install elasticsearch


#### 2.1安装logstash见官网： ####
https://www.elastic.co/guide/en/logstash/current/installing-logstash.html
确认logstash是否安装好
切换到logstash/bin目录如：
cd logstash-6.2.4（使用centos yum安装logstash目录在/usr/share/logstash/bin）
bin/logstash -e 'input { stdin { } } output { stdout {} }'
查看输出

#### 2.2添加用户： ####

切换到elasticearch安装目录执行一下命令
```
bin/x-pack/users useradd my_admin -p my_password -r superuser
```



#### 3.创建index此处略过。。。 ####


#### 4.下载mysql驱动解压到某个目录这个目录需要配置到步骤4 ####


#### 5.配置mysql.conf并放到目录/usr/share/logstash/bin/config-mysql/mysql.conf ####
mysql.conf内容
```
input {
    stdin{
    }
    jdbc {
      # 数据库
      jdbc_connection_string =&gt; "jdbc:mysql://localhost:3306/mydatabase"
      jdbc_user =&gt; "root"
      jdbc_password =&gt; "数据库密码"
      # mysql驱动解压的位置
      jdbc_driver_library =&gt; "/usr/share/logstash/data/mysql-connector-java-5.1.46/mysql-connector-java-5.1.46-bin.jar"
       # mysql驱动类
      jdbc_driver_class =&gt; "com.mysql.jdbc.Driver"
      jdbc_paging_enabled =&gt; "true"
      jdbc_page_size =&gt; "50000"
      #可以使用sql文件的方式
      #statement_filepath =&gt; "config-mysql/tag.sql"
      #要同步的表
      statement =&gt; "SELECT * from tbl_tag"
      schedule =&gt; "* * * * *"
      #索引type
      type =&gt; "tag"
      #索引设置时区
      jdbc_default_timezone =&gt; "Asia/Shanghai"
    }
}
 
 
output {
    elasticsearch {
        hosts =&gt; "localhost:9200"
	#索引名称
        index =&gt; "myindex"
        document_id =&gt; "%{id}"
        user =&gt; elastic用户名
        password =&gt; "elastic登录密码"
	#输出的索引type此处表示上面的tag
       "document_type" =&gt; "%{type}"
    }
    stdout {
        codec =&gt; json_lines
    }
}
```

#### 6.切换使用命令查看是否成功 需要root用户 ####
```
./logstash -f config-mysql --path.data=/root/
```

作者：天空蓝蓝的  www.lskyf.com   www.lskyf.xyz  
版权所有，欢迎保留原文链接进行转载：)


