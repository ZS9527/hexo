---
title: 排查本地写入sql文件太慢
date: 2020-11-07 15:04:35
tags: Mysql
---
# 排查本地写入sql文件太慢
> 记录一下日常
> 本地写入数据每分钟2000条，所以叫大佬帮忙排查一下。
> 主要原因 binlog 日志、主从复制。
> 

<!--more-->
## 查看磁盘情况
首先怀疑是否是磁盘出了问题，打开任务管理器 -> 性能 -> 资源管理器（或者直接运行命令`perfmon.exe` 打开）。
查看磁盘情况，发现磁盘已经跑满。写入排行里有我的两个 mysql 服务和 binlog 日志服务。
由于本机主从复制只是测试用，所以可以关闭。
## 关闭主从复制
直接在从库中输入命令即可。
```
stop slave;
```
## 关闭 binlog 日志
在主库的 my.ini 配置文件里注释配置
```
#log-bin=mysql-bin
#binlog_format=mixed
#server-id   = 1
#expire_logs_days = 10
```
在服务里重启 mysql 服务。

成功解决问题。
