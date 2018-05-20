---
layout: post
title: 使用logstash从 mysql 全量同步数据到elasticsearch
tags:
- mysql
- elasticsearch
- logstash


- 使用logstash从 mysql 全量同步数据到elasticsearch
categories: elasticsearch
description:  mysql elasticsearch 全量同步数据
---
## 使用logstash从 mysql 全量同步数据到elasticsearch
   mysql elasticsearch 全量同步数据
<!-- more -->

使用logstash从 mysql 全量同步数据到elasticsearch
## 1.	 安装elasticsearch,安装logstash
#### (1)elasticsearch此处略过,安装logstash见官网：https://www.elastic.co/guide/en/logstash/current/installing-logstash.html
确认logstash是否安装好
切换到logstash/bin目录如：
cd logstash-6.2.4（使用centos yum安装logstash目录在/usr/share/logstash/bin）
bin/logstash -e 'input { stdin { } } output { stdout {} }'
查看输出 
 
#### (2)创建index此处略过。。。 
#### (3) 下载mysql驱动解压到某个目录这个目录需要配置到步骤4
#### (4)配置mysql.conf并放到目录/usr/share/logstash/bin/config-mysql/mysql.conf
mysql.conf内容


	input {
		stdin{
		}
		jdbc {
		  # 数据库
		  jdbc_connection_string => "jdbc:mysql://localhost:3306/mydatabase"
		  jdbc_user => "root"
		  jdbc_password => "数据库密码"
		  # mysql驱动解压的位置
		  jdbc_driver_library => "/usr/share/logstash/data/mysql-connector-java-5.1.46/mysql-connector-java-5.1.46-bin.jar"
		   # mysql驱动类
		  jdbc_driver_class => "com.mysql.jdbc.Driver"
		  jdbc_paging_enabled => "true"
		  jdbc_page_size => "50000"
		  #可以使用sql文件的方式
		  #statement_filepath => "config-mysql/tag.sql"
		  #要同步的表
		  statement => "SELECT * from tbl_tag"
		  schedule => "* * * * *"
		  #索引type
		  type => "tag"
		  #索引设置时区
		  jdbc_default_timezone => "Asia/Shanghai"
		}
	}

	output {
		elasticsearch {
			hosts => "localhost:9200"
		#索引名称
			index => "myindex"
			document_id => "%{id}"
			user => elastic用户名
			password => "elastic登录密码"
		#输出的索引type此处表示上面的tag
		   "document_type" => "%{type}"
		}
		stdout {
			codec => json_lines
		}
	}
	
	
#### (5) 切换到bin目录使用命令查看是否成功 需要root用户
./logstash -f config-mysql --path.data=/root/
