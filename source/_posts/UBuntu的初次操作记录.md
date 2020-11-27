---
title: Ubuntu的初次操作记录
date: 2020-09-05 14:20:13
tags: Ubuntu
---

# Ubuntu的初次操作记录
> 简单的记录一下操作过程，做个存档。主要目的是排查服务器 ssh 关闭，并安装切片程序。
> 包含防火墙检查，ssh安装，更换数据源，解决域名无法解析。
> 
<!--more-->

## 检查防火墙
拿到服务器后首先检查了一下防火墙是否开启。
```
sudo ufw status
```
发现防火墙已经关闭。

## 检查ssh是否安装
检查这个机器是否安装了 ssh-server 服务，一般都是只安装了 client。
```
dpkg -l | grep ssh
```
发现并没有安装，使用命令开始安装。
```
sudo apt-get install openssh-server
```
报错 ubuntu 数据源无法下载。准备更换国内的阿里数据源。

## 更换数据源之前
这个服务器不能解析域名，我还是在下面的步骤中的第四步执行时，发现无法解析。
修改 resolv.conf 文件
```
sudo gedit /etc/resolv.conf 
```
打开后发现整个文件没有任何配置，在最后一行加入。
```
nameserver 8.8.8.8
```

## 更换 apt 默认数据源为阿里
这里就不得不吐槽一下，拿到得是一个从 ie 浏览器访问 vmware 管理界面，然后再从管理界面访问我们的机器。
这就导致无法复制命令，下面的数据源更换愣是靠手打的。
1. 备份源列表
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```
2. 命令行打开sources.list文件
```
sudo gedit /etc/apt/sources.list
```
3. 修改sources,list文件【本例更改为阿里镜像源】
```
#  阿里镜像源

deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```
4. 更新并升级
```
sudo apt-get update && sudo apt-get upgrade
```
在输入了这个更新命令后，等待了一段时间。其中有很多的忽略包，不知道后面会不会需要重新弄。

## 结束
更换完数据源后，即可成功安装 ssh。