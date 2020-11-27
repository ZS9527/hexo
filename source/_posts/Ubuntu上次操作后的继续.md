---
title: Ubuntu上次操作后的继续操作
date: 2020-09-12 15:53:10
tags: Ubuntu
---

# Ubuntu上次操作后的继续操作
> 上次搞完哪个服务器后，再次使用发现无法用 sftp 传输文件。
> 包含查询系统版本号，更换清华软件数据源，处理文件无法上传。
<!--more-->

## apt 无法定位软件包
当我使用服务器安装 gcc-c++ 时，发现报错。
查询后建议换下载数据源。
(https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)[https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/] 通过这个地址来试图更换为清华数据源。
查询 Ubuntu 版本信息
```
cat /proc/version
```
显示
```
Linux version 4.15.0-45-generic (buildd@lcy01-amd64-027) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.10)) #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019

```
可以选择对应版本的数据源来更换。
但是，通过 Xshell 直接操作时发现 sftp 无法传递文件。

## sftp 无法传递文件
主要问题是可以下载，但是不能上传。首先查看用户权限，发现登录的用户权限不足。无法修改文件夹，只能用命令修改权限后上传。
```
chmod 777 /etc/apt/sources.list
```
