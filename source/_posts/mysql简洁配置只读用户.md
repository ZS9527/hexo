---
title: mysql简洁配置只读用户
date: 2020-05-15 16:06:22
tags: mysql
---

# mysql简洁配置只读用户
> 这里只是记录一下自己对数据库的操作。之后可以直接重新使用。

<!--more-->

## 过程
删除已有用户，之前配置错误。需要重新来。
```
drop user 'qw'@'%';
```
配置用户
```
CREATE USER 'qw'@'%' IDENTIFIED BY 'qw123456';
```
分配只读权限，前面的\*是库名。后面的\*是表名
```
GRANT select ON *.* TO 'qw'@'%';
```
刷新权限
```
flush privileges;
```