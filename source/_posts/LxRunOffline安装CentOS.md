---
title: LxRunOffline安装CentOS
date: 2020-08-01 20:00:32
tags: LxRunOffline
---

# LxRunOffline安装CentOS
> 首先，本机已经是 windows2004 版本，安装了阿里云提供的[Docker for Windows](http://mirrors.aliyun.com/docker-toolbox/windows/docker-for-windows/) 。
> 本文承接上文的《在win上使用VMware安装虚拟机运行tippecanoe切片》,本质都是在 Windows10 上安装Linux虚拟机。
> 
<!--more-->

## 安装LxRunOffline
- 普通手动安装：下载解压 [LxRunOffline](https://github.com/DDoSolitary/LxRunOffline/releases) ，并设置环境变量。
- 使用 [Chocolatey](https://chocolatey.org/) （windows 中的一个软件包管理工具）安装。
```
choco install lxrunoffline
```
- 使用 [Scoop](https://scoop.sh/) （windows 中的一个软件包管理工具） 安装。
```
scoop bucket add extras
scoop install lxrunoffline
```

我本人使用的是第一种，下载解压后把`LxRunOffline.exe`的本地路径（例：`D:\LxRunOffine\LxRunOffline.exe` ）放在 Path 环境变量中。

## 下载Centos镜像
直接下载下面的链接
```
https://raw.githubusercontent.com/CentOS/sig-cloud-instance-images/a77b36c6c55559b0db5bf9e74e61d32ea709a179/docker/centos-7-docker.tar.xz
```
## 使用 LxRunOffline 部署 Centos 到WSL
```
LxRunOffline.exe  install -n centos -d E:\ProgramData\Microsoft\Windows\WSL\CentOS -f  E:\Progra
mData\Microsoft\Windows\WSL\centos-7-docker.tar.xz
```
其中 -d 后面是要安装到的目录，-f 是前面下载的镜像， -n 用来指定名称。

使用微软应用商店安装的 WSL 会在开始菜单添加应用图标（快捷方式），而使用 LxRunOf­fline 安装 WSL 时可以通过添加 -s 参数在桌面创建快捷方式。如果你安装时忘记添加参数，可以使用以下命令进行创建。
```
LxRunOffline s -n <WSL名称> -f <快捷方式路径>.lnk
```

然后使用  LxRunOffine 来开启 Centos

```
LxRunOffline  run  -n centos
```

## 在 Centos 中部署 tippecanoe
更新yum配置
```
yum install gcc-c++
yum install sqlite-devel
yum install -y zlib zlib-devel
```
下载解压 [tippecanoe](https://github.com/mapbox/tippecanoe) 包，在解压后的文件夹里
```
make && make install
```

如果碰到`-bash: make: command not found`这个错误，安装make命令即可
```
yum -y install gcc automake autoconf libtool make
```
