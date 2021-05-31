##1.配置docker-compose
### 1.1.上传主docker-compose.yml注意:复制的时候去掉注释和中文，主要配置server-id=1，read-only=0，端口号为3306
docker-compose.yml
```yaml
version: '3.1'
services:
  db:
    image: mysql
    container_name: master
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root  #root用户密码
      TZ: Asia/Shanghai
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --log-bin=mysql-bin #开启日志
      --sync_binlog=1
      --server-id=1  #指定ID
      --read-only=0  #master的read-only=0，slave的read-only=1
      --binlog-ignore-db=sys #要忽略的数据库
      --binlog-ignore-db=mysql
      --binlog-ignore-db=sys 
      --binlog-ignore-db=performation_schema 
      --binlog-do-db=yaunfenge_db1 #要同步的数据库
      --binlog-do-db=yaunfenge_db2 #要同步的数据库
    ports:
      - 3306:3306 
    volumes:
      - ./data:/var/lib/mysql
      - ./conf/my.cnf:/etc/my.cnf
      - ./log:/var/log/mysql
```

### 1.2.从库docker-compose.yml注意:复制的时候去掉注释和中文，配置server-id=2，read-only=1 （这里是和master的区别），端口号为3309

```yaml
version: '3.1'
services:
  db:
    image: mysql
    container_name: slave01
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      TZ: Asia/Shanghai
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --log-bin=mysql-bin 
      --sync_binlog=1
      --server-id=2 
      --read-only=1 
      --binlog-ignore-db=sys
      --binlog-ignore-db=mysql
      --binlog-ignore-db=sys 
      --binlog-ignore-db=performation_schema 
      --binlog-do-db=yaunfenge_db1
    ports:
      - 3309:3306 
    volumes:
      - ./data:/var/lib/mysql
      - ./conf/my.cnf:/etc/my.cnf
      - ./log:/var/log/mysql
```
	  
### 1.3.在docker-compose.yml执行docker-compose up -d 启动


## 2.加入集群
### 2.1 主服务器创建同步用户并授权

```bash
#通过Navicat中创建用户并授权
CREATE USER 'backup'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE ON *.* TO 'backup'@'%';
FLUSH PRIVILEGES;
#查看master的binlog信息
SHOW MASTER STATUS ;
得到 File mysql-bin.000003下面用到
```
### 2.2 从库slave01加入集群

从机slave01,slave0,slave.....n...
启动slave容器，实现主从同步
#### 2.2.1 启动容器,等待mysql正常启动

#### 2.2.2 通过Navicat连接slave执行下面命令配置  MASTER_HOST , MASTER_USER , MASTER_LOG_FILE , MASTER_PORT
```bash
CHANGE MASTER TO MASTER_HOST='192.168.56.120',MASTER_USER='backup',MASTER_PASSWORD='123456',MASTER_LOG_FILE='mysql-bin.000003',MASTER_LOG_POS=0,MASTER_PORT=3306;
```

#### 2.2.3 开启同步
```bash
START SLAVE;
```
#### 2.2.4 查看slave同步状态
```bash
SHOW SLAVE STATUS 
```

如果 Slave_IO_Running 是Yes并且Slave_Sql_Running 那么就成功了

### 2.3 其他注意事项
#### 2.3.1 如果Slave需要修改先或有什么问题先执行
```
STOP SLAVE;
RESET SLAVE; 
```

再执行以下代码重新加入
```bash
CHANGE MASTER TO MASTER_HOST='192.168.56.120',MASTER_USER='backup',MASTER_PASSWORD='123456',MASTER_LOG_FILE='mysql-bin.000003',MASTER_LOG_POS=0,MASTER_PORT=3306;

START SLAVE;

SHOW SLAVE STATUS;
```

#### 2.3.2 如果master需要修改或有什么问题先执行 RESET MASTER在进行其他步骤;

## 3.测试
###将以下sql语句执行后
```sql
 create DATABASE db_yuanfenge;
 CREATE TABLE `tbl_student` (
  `id` int NOT NULL,
  `name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ;
INSERT INTO `db_yuanfenge`.`tbl_programmer`(`id`, `name`) VALUES (1, '猿份哥');
```

## 4.说明
上文的logbin参数都配置在docker-compose.yml里了，也可以根据挂载的路径将logbin信息配置到./conf/my.cnf文件里