---
title: linux服务器搭配win7客户端的frp
date: 2019-12-07 10:36:46
tags: frp
---

# linux服务端搭配win7客户端的frp
> teamviewer安装是方便一些，但是频繁的提示商业用途和不停的掉线太影响使用了。
> 我还是重新找了一个方法来远程电脑。

<!--more-->

## 下载frp
从[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases) 下载最新版本的frp压缩包。因为是两种系统，所以把linux和windows的版本都下载了一份。

## linux服务端
使用WinSCP工具将压缩包传到了具有公网ip的云服务器上。解压后修改里面的frps.ini。
```
[common]
bind_port = 7000           #与客户端绑定的进行通信的端口
```
保存后启动服务./frps -c ./frps.ini。
### 注意
查看一下防火墙，记得打开7000端口。
#### 使用systemctl来控制自启动
参考[Frp后台自动启动的几个方法](https://blog.csdn.net/x7418520/article/details/81077652)这篇博客配置
这个方法比较好用，很方便 
```
sudo vim /lib/systemd/system/frps.service 
```

在frps.service里写入以下内容
```
[Unit]
Description=fraps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
#启动服务的命令（此处写你的frps的实际安装目录）
ExecStart=/your/path/frps -c /your/path/frps.ini

[Install]
WantedBy=multi-user.target
```
然后就启动frps 
```
sudo systemctl start frps 
```
再打开自启动 
```
sudo systemctl enable frps
```

如果要重启应用，可以这样，**sudo systemctl restart frps**
如果要停止应用，可以输入，**sudo systemctl stop frps**
如果要查看应用的日志，可以输入，**sudo systemctl status frps**

## windows客户端
解压后修改其中的frpc.ini
```
[common]
server_addr = 服务器ip
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 3389 远程桌面端口
remote_port = 7001 访问时的端口
```
保存后启动服务
```
frpc.exe -c frpc.ini
```
### 注意
这里可以配置成开机自动启动。
做一个.bat文件，文件内容：
```
C:\frp_0.30.0_windows_amd64\frpc.exe -c C:\frp_0.30.0_windows_amd64\frpc.ini
```
再从任务计划程序中设置启动时运行文件即可。