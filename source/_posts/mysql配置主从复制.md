---
title: mysql配置主从复制
date: 2019-09-20 16:14:05
tags: mysql
---

# mysql配置主从复制
> 本文来自[https://www.cnblogs.com/gl-developer/p/6170423.html](https://www.cnblogs.com/gl-developer/p/6170423.html), 仅仅为了下次使用做一个备份。

<!--more-->

## 配置过程
- 主服务器：
	- 开启二进制日志
	- 配置唯一的server-id
	- 获得master二进制日志文件名及位置
	- 创建一个用于slave和master通信的用户账号
- 从服务器：
	- 配置唯一的server-id
	- 使用master分配的用户账号读取master二进制日志
	- 启用slave服务
### 一丶主数据库master修改：
**1.**修改mysql配置
找到主数据库的配置文件my.cnf(或者my.ini)，在[mysqld]部分插入如下两行：
```xml
[mysqld]
log-bin=mysql-bin #开启二进制日志
server-id=1 #设置server-id

# 不同步哪些数据库  
binlog-ignore-db = mysql  
binlog-ignore-db = test  
binlog-ignore-db = information_schema  
  
# 只同步哪些数据库，除此之外，其他不同步  
binlog-do-db = game  
```
**2.**重启mysql，创建用于同步的用户账号
打开mysql命令行

```sql
 mysql -u root -p --protocol=tcp --host=localhost --port=3308（选择端口）
 use mysql;
 CREATE USER 'repl'@'123.57.44.85' IDENTIFIED BY 'slavepass';#创建用户
 GRANT REPLICATION SLAVE ON *.* TO 'repl'@'123.57.44.85';#分配权限
 flush privileges;   #刷新权限
```
**3.**查看master状态，记录二进制文件名(mysql-bin.000003)和位置(73)：

```
SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 | 73       | test         | manual,mysql     |
+------------------+----------+--------------+------------------+
```
### 二丶从服务器slave修改：
**1.**修改mysql配置
同样找到my.cnf配置文件，添加server-id

```xml
server-id=2 #设置server-id，必须唯一
```
**2.**重启mysql，打开mysql会话，执行同步SQL语句(需要主服务器主机名，登陆凭据，二进制文件的名称和位置。其中的端口号是主库)：

```sql
mysql> CHANGE MASTER TO
    ->     MASTER_HOST='182.92.172.80',
    ->     MASTER_USER='repl',
    ->	   MASTER_PORT=3306,
    ->     MASTER_PASSWORD='slavepass',
    ->     MASTER_LOG_FILE='mysql-bin.000003',
    ->     MASTER_LOG_POS=73;
```
**3.**启动slave同步进程：

```sql
start slave;
```
**4.**查看slave状态：
```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 182.92.172.80
                  Master_User: rep1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000013
          Read_Master_Log_Pos: 11662
               Relay_Log_File: mysqld-relay-bin.000022
                Relay_Log_Pos: 11765
        Relay_Master_Log_File: mysql-bin.000013
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
```
当Slave_IO_Running和Slave_SQL_Running都为YES的时候就表示主从同步设置成功了。
