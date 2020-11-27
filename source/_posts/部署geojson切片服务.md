---
title: 部署geojson切片服务
date: 2020-08-08 20:42:27
tags: Geojson
---

# 部署geojson文件矢量切片服务
> 客户答应的 Linux 虚拟机到了，终于可以直接部署 tippecanoe 了。
> 原本的 Geojson 文件太大了，还是需要切片。

## 安装 tippecanoe
登录 Linux 服务器
1. 准备好 tippecanoe 源码。
2. 将服务器的 yum 源更改为 阿里源
3. 下载依赖
```
yum install gcc-c++
yum install sqlite-devel
yum install -y zlib zlib-devel
```
4. 将源码解压好后，在 tippecanoe 文件夹内
```
make -j
make install
```
完成
具体的命令行参数参考：[https://github.com/mapbox/tippecanoe](https://github.com/mapbox/tippecanoe)
## 安装 FoxGIS Server Lite
直接部署在了原本的 win 服务器上，通过 FTP 读取 tippecanoe 做好的矢量地图文件。

通过[https://jingsam.github.io/foxgis-server-lite/#/deploy](https://jingsam.github.io/foxgis-server-lite/#/deploy) 下载 win 包，直接启动 exe 文件即可。

## 部署调用命令 jar 包
这个程序是负责调用 Linux 中的切片程序，只需要放在 Linux 任意文件夹中开启来就完事了。只是一个开发接口调用命令而已。

## 处理中间关联程序
切片程序部署在 Linux 服务器上，将原本的 geojson 文件通过 FTP上传到 Linux 文件夹中。
再访问接口来启动切片命令，收到完成回复后，将 FTP 中切好的文件拿回 win 服务器。
放在 FoxGIS Server Lite 中，得到映射的 url 。

注意中间我关闭了 Linux 服务器的防火墙和 SeLinux，部署 FTP 的方式是用的之前的文章。

