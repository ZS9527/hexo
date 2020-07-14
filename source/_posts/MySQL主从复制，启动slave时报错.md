---
title: MySQL主从复制，启动slave时报错Slave failed to initialize relay log info structure from the repository
date: 2019-12-21 15:11:53
tags: mysql
---

# MySQL主从复制，启动slave时报错Slave failed to initialize relay log info structure from the repository

> 本机的主从同步再次崩溃了。

<!--more-->

## 报错信息
mysql> start slave; 
ERROR 1872 (HY000): Slave failed to initialize relay log info structure from the repository

## 处理流程
查看了一下错误日值
```
Failed to open the relay log '.\WIN-DOUOFQR7DBP-relay-bin.000040' (relay_log_pos 333357).

Could not find target log file mentioned in relay log info in the index file '.\myhbase-relay-bin.index' during relay log initialization.
```

在百度中找到一个类似的文章[https://blog.csdn.net/weixin_37998647/article/details/79950133](https://blog.csdn.net/weixin_37998647/article/details/79950133) 

使用命令
```
mysql> reset slave;
Query OK, 0 rows affected (0.01 sec)

mysql> change master to ......

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```
即可成功解决问题。