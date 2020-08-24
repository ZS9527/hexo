---
title: 在win上使用VMware安装虚拟机运行tippecanoe切片
date: 2020-07-31 19:21:48
tags: linux
---

# 在win上使用VMware安装虚拟机运行tippecanoe切片
> 记录自己安装 tippecanoe 的过程
> tippecanoe 为生成本地矢量瓦片的开源工具，介绍文章在[Mapbox GL JS学习笔记四：开源工具生成本地矢量瓦片](https://zhuanlan.zhihu.com/p/31185974) 、[Mapbox GL JS学习笔记七：mb-util工具使用](https://zhuanlan.zhihu.com/p/31760283)。
> 步骤为使用 VMware 安装 Linux 虚拟机，将 tippecanoe 部署在虚拟机上。再将启动服务部署在虚拟机上，使用命令调用切片。用另外的 FoxGIS Server Lite 部署在 Windows 主机上来访问切片后的瓦片文件
> 参考文章：[FoxGIS安装部署](https://jingsam.github.io) 、[Mapbox GL JS 学习笔记八：tippecanoe在centos上安装以及使用补充说明](https://zhuanlan.zhihu.com/p/32306317)
> 在安装过程中出现错误，无法继续在本机安装。

<!--more-->

## 1.安装VMware虚拟机
从官网（[试用 VMware Workstation Pro](https://www.vmware.com/cn/products/workstation-pro/workstation-pro-evaluation.html)）下载最新版 VMware 。

在[http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso](http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso) 下载好 iso 镜像文件。

一路下一步完成 VMware 的安装，创建新的虚拟机。使用 centos7 的镜像包完成安装。

## 2.在centos上安装tippecanoe
安装完后尝试 ping 114 ，发现报错
```
connect: Network is unreachable 
```

使用[https://blog.csdn.net/yohjob/article/details/90724914](https://blog.csdn.net/yohjob/article/details/90724914) 这个博客的方法解决了。
### 2.1.解决centos系统网络不通问题
1. 进入网卡配置文件所在目录：
```
cd /etc/sysconfig/network-scripts/
```

2. 编辑网卡配置文件
```
vi ifcfg-ens33
```

3. 将文件中的最后一行的ONBOOT=no 改为ONBOOT=yes

4. 保存文件并退出vi，重启network
```
service network restart
```

### 2.2.安装VMtool
这个工具安装一下还是很方便的，可以操作的更加流畅。

1. 选择 虚拟机 >> 客户机 >> 安装/升级vmwareTools
2. 装载cd(挂载到media文件夹下)：
```
mount /dev/cdrom /media
提示mount:block device /dev/sr0 is write-protecter, mounting read-only

#找到挂载好的压缩包
cd VMware\ Tools/
ls
#将其复制到tmp中
cp VMwareTools-9.0.0. (按Tab补全) /tmp
cd /tmp

#授权并解压
chmod +x VMwareTools-8.6.0-425873.tar.gz
tar zxf VMwareTools-8.6.0-425873.tar.gz

#安装
cd vmware-tools-distrib/
./vmware-install.pl
```
3. 如果出现`bash:./vmware-install.pl :/usr/bin/perl:bad interpreter:No such file or directory`，输入`yum groupinstall "Perl Support"`即可。
4. 一路回车，出现重复现象时注意翻译一下后选择yes或no

### 2.3.安装tippecanoe
1. 先更新依赖
```
yum install gcc-c++
yum install sqlite-devel
yum install -y zlib zlib-devel
```

2. 挂载虚拟机
将虚拟机关机后，选择编辑虚拟机设置。
![b1.png](b1.png)

在虚拟机设置中选择选项->共享文件夹
![b2.png](b2.png)

选择下一步
![b3.png](b3.png)

填写文件夹路径
![b4.png](b4.png)

完成
![b5.png](b5.png)

```
/usr/bin/vmhgfs-fuse .host:/vmwareGeojson /mnt/hgfs -o subtype=vmhgfs-fuse,allow_other
/usr/bin/vmhgfs-fuse .host:/vmwareGeojson /mnt/hgfs -o subtype=vmhgfs-fuse,allow_other
```

3. 出现问题
用 VMware 安装的 Centos7 进行到这里时无法共享文件夹，参考同事的方法用 virtualBox 安装的Centos7 无法开机。所以放弃这种方式运行。