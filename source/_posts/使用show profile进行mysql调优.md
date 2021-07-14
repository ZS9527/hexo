---
title: 使用show profile进行mysql调优
date: 2020-12-12 17:01:10
tags: Mysql
---

# 使用show profile进行mysql调优
> 本文首先参考《高性能 MySQL（第三版）》中的第3章第3节剖析MySQL查询
> 
<!--more-->

## 问题来源
在日常开发接口的过程中，有一个接口在开发上线后，偶尔会发生接口查询超时问题。一开始是服务器部署网络会有网络波动，突发性的网络延迟。便把问题搁置，归结为网络问题。

过了近两个月后，接口超时现象变得频繁了起来。需要仔细查看原因。

## 问题解决过程
在程序中埋日志，打印时间，去缓存等操作后，定位到一个分表的查询会出现超时现象。由于语句过于简单，一共查询结果最大为120条。本人没有想到优化方法，向大佬求助。

大佬去排查了MySQL 慢语句日志，这里说明一下慢查询日志。

### 慢查询日志
根据本文参考书籍来讲，慢查询是为了让MySQL记录下查询超过指定时间的语句，通过定位分析性能的瓶颈，才能更好的优化数据库系统的性能。书籍作者描述慢查询日志在I/O密集型场景中开销可忽略不计，只需要担心消耗的磁盘空间。

**开启方式：** 在配置文件中配置
```sql
slow_query_log=1 #慢查询开启状态，1开启，0关闭 默认为0
slow_query_log_file=“slow.log” #慢查询日志存放的位置（这个目录需要MySQL的运行帐号的可写权限，一般设置为MySQL的数据存放目录） 默认给一个缺省的文件host_name-slow.log
long_query_time=10  #查询超过10秒才记录 默认为10
```
在开启的日志中，查询到了 SQL 语句的执行时间为16秒。开始进一步调查。

### show profile
大佬有别的事被抓走了，我只好自己去查询资料来寻找定位问题方法。
继续阅读《MySQL高性能》，尝试使用其中的 `show profile`方法。
**显示当前mysql是否支持profile：**`select @@have_profiling`
**查看profile是否开启：**`select @@profiling`如果为0是未开启
**开启方式：**`set profiling =1`
当开启后，就可以进入数据库执行语句，来查看分析语句的执行结果。
```sql
use test

select * from test
```
大概是这样的语句执行完毕后，就可以查看语句的执行结果。
**查看结果：**`show profiles;`
![b1.png](show profiles)
获得了语句的执行编码后，可以直接查看对应编码的语句详细执行时间
**查看对应编码：**`show profile for query query_id`
其中可以加入参数来查看对应的消耗时间。
	1. ALL：显示所有的开销信息。
	2. BLOCK IO：显示块IO开销。
	3. CONTEXT SWITCHES：上下文切换开销。
	4. CPU：显示CPU开销信息。
	5. IPC：显示发送和接收开销信息。
	6. MEMORY：显示内存开销信息。
	7. PAGE FAULTS：显示页面错误开销信息。
	8. SOURCE：显示和Source_function，Source_file，Source_line相关的开销信息。
	9. SWAPS：显示交换次数开销信息。
![b2.png](show profile for query)

#### 日常开发需注意的结论
①converting  HEAP to MyISAM：查询结果太大，内存不够，数据往磁盘上搬了。

②Creating tmp table：创建临时表。先拷贝数据到临时表，用完后再删除临时表。

③Copying to tmp table on disk：把内存中临时表复制到磁盘上，危险！！！

④locked。

如果在show profile诊断结果中出现了以上4条结果中的任何一条，则sql语句需要优化。

### 后续解决步骤
之后调查到程序的cpu时间超长，但是磁盘的时间未显示。
查看资源监视器，发现磁盘写入100%。
尝试将同服务器的写入减少，把java.io.FileWriter 换成 java.nio.file.Files.write。
