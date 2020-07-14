---
title: mysql错误：Incorrect key file for table
date: 2019-11-30 10:39:13
tags: mysql
---

# mysql错误：Incorrect key file for table
> 此次错误有多种，首先是我本机的**Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'**。
> 然后是服务器的**Incorrect key file for table '.\env_v2_air\gather_air_data_201911.MYI'; try to repair it**

<!--more-->

## 本机错误
### 发生场景：
本机的mysql一开始使用5.7版本作为主库，8.1版本作为从库。这次主从同步崩溃后，尝试使用复制data文件夹的方式已经不能够把表复制到从库，而且C盘的空间要不足了。我便尝试将主从库搬迁到剩余空间比较多的F盘。
经过了二天的sql数据导出和导入，我将原主库的表成功写入了新的主库和从库里。磁盘的读写速度是限制了这次sql数据迁移效率的最大问题，平均1秒几十条的速度就导致磁盘读写占用90%。日常软件的使用都收到了影响。
当我配置主从同步的时候，从库的状态出现了错误：**Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file**
### 处理方法：
#### 第一次尝试：
发现没有配置主库的端口，配置端口后start slave可以成功显示两个yes。但是却发现表不同步，于是重新复制主库的文件。
#### 第二次尝试：
复制主库的data文件后，start slave报错。**Slave failed to initialize relay log info structure from the repository**。使用网上一篇文章的操作[Slave failed to initialize relay log info structure from the repository, Error_code: 1872](https://blog.csdn.net/lwei_998/article/details/41210945)。重新reset slave,并没有在my.ini中添加relay_log配置。即可重新开启主从复制


## 服务器错误
### 发生场景：
服务器异常关机后，重新启动程序时报错：**Incorrect key file for table '.\v_v_a\g_a_data_201911.MYI'; try to repair it**。
主从同步也报错：
```
Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.
```

### 处理方法：
#### 第一次尝试：
首先运行
```
check table g_a_data_201911;
```
然后
```
repair table g_a_data_201911;
```
再my.ini加入**relay_log_recovery=1**重新启动mysql，主从同步报错更换
```
Could not execute Write_rows event on table env_v2_air.gather_air_data_201911; Incorrect key file for table '.\env_v2_air\gather_air_data_201911.MYI'; try to repair it, Error_code: 126; Incorrect key file for table '.\env_v2_air\gather_air_data_201911.MYI'; try to repair it, Error_code: 126; Incorrect key file for table 'gather_air_data_201911'; try to repair it, Error_code: 1034; handler error HA_ERR_CRASHED; the event's master log mysql-bin.000211, end_log_pos 547782372
```
#### 第二次尝试：
在从库里重复运行上面的命令，检查数据是否一致后。重新开启主从复制，成功。

## 后记：
因为时间关系，并没有详细的错误原理说明。
