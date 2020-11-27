---
title: Ubuntu安装tippecanoe
date: 2020-09-19 17:24:17
tags: tippecanoe
---

# Ubuntu安装tippecanoe
> Ubuntu 的命令还是和 centos 有很多出入，没法直接复制。再次整理一份安装过程。
> 参考博客 [https://blog.csdn.net/zxzfcsu/article/details/80836277](https://blog.csdn.net/zxzfcsu/article/details/80836277)
<!--more-->

## 安装依赖
安装 gcc
```
sudo apt-get  install  build-essential
```

安装 sqlite-devel
```
sudo apt-get install sqlite sqlite3
```
同时还要安装 ruby
```
sudo apt-get install ruby
```

安装 zlib zlib-devel
```
sudo apt-get install zlib1g
sudo apt-get install zlib1g.dev
```

后来在 `make -j` 的时候还报错`fatal error:sqlite3.h` 。又安装了
```
sudo apt-get install libsqlit3-dev
```
并没有报`fatal error:zlib.h` 就没有使用
```
sudo apt-get install zlib1g-dev
```
## 解压
剩下的步骤就是一样的了，解压后运行命令即可。
```
make -j
make install
```