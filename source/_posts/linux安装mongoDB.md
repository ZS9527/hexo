---
title: linux通过docker安装mongoDB
date: 2020-03-28 19:44:09
tags: linux
---

# linux通过docker安装mongoDB
> 记录一下安装过程，之后再次安装时用。

<!--more-->

## 前期准备
拿到服务器后先检查一下内核版本
```
uname -r

cat /etc/redhat-release
```
再更新一下yum数据源
```
# 备份系统自带yum源配置文件
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 进入yum源配置文件所在的文件夹
cd /etc/yum.repos.d/

#163的yum源配置文件 
# CentOS7
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
# CentOS6
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
# CentOS5
wget http://mirrors.163.com/.help/CentOS5-Base-163.repo

#ailiyun的yum源配置文件
# CentOS7
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# CentOS6
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
# CentOS5

# 选择上面合适的数据源后，生成缓存
yum makecache
```

## 安装docker（包含重新安装docker，更换版本）
使用命令
```
yum install -y docker
```

查看docker是否安装成功
```
yum list installed |grep docker
```

启动docker服务并设置docker开机自启
```
systemctl start docker.service
systemctl enable docker.service
```
### 错误开始
在启动时报了一个错误，让我去看一下日志
```
systemctl status docker
```
看到了这样的错误
```
Error starting daemon: SELinux is not supported with the overlay2 graph driver...false)
```
尝试卸载重装docker，依旧不能解决问题
```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
查找了一下，需要修改docker配置文件
```
vi /etc/sysconfig/docker
```
打开后修改里面的内容，本来只有一个`--selinux-enabled`，在后面加上`=false`，就可以解决
```
--selinux-enabled=false
```
### 错误解决后
查看docker版本
```
docker version
```
配置docker镜像加速器（阿里），打开[阿里云](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors?accounttraceid=fd4b2b1718054019ba8f49c3dcc68f62byrh) 登录，按照教程配置
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [""]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 重新安装docker
依旧是用之前卸载方式卸载docker，然后恢复yum数据源。重新配置为centos7的阿里云源。
再增加阿里docker镜像
```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
安装依赖包
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```
刷新yum缓存，重新安装
```
yum install docker-ce -y
```
用命令``出现错误
```
failed to start daemon: error initializing graphdriver: driver not supported
```
按照百度后的方法
```
rm -rf /var/lib/docker/*
```
可以成功启动
```
systemctl status docker
docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: active (running) since 五 2020-04-04 15:25:46 CST; 37s ago
     Docs: https://docs.docker.com
 Main PID: 32067 (dockerd)
   Memory: 46.5M
   CGroup: /system.slice/docker.service
           └─32067 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.soc
```
查看docker版本，配置阿里云镜像，继续安装mongoDB
```
docker version

Client: Docker Engine - Community
 Version:           19.03.8
 API version:       1.40
 Go version:        go1.12.17
 Git commit:        afacb8b
 Built:             Wed Mar 11 01:27:04 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.8
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.12.17
  Git commit:       afacb8b
  Built:            Wed Mar 11 01:25:42 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683

```

## 安装mongoDB
查询一下镜像
```
docker search mongo
```
安装镜像
```
docker pull mongo
```
但是再次出现了bug
```
Error response from daemon: error creating overlay mount to xxx merged: invalid argument
```
百度一下发现都是删除镜像后将overlay2文件换成overlay文件系统。仔细思考后决定还是先重装docker版本。

重装后可以正常安装镜像，再次启动容器，成功（注意：没有解决服务器映射端口问题，直接用host模式）
```
docker run -d -it --net=host -v /home/mongoDB:/data/db --name docker_mongodb  mongo
```
紧接着就是端口无法访问的问题了。防火墙已经打开端口，给linux安装好telnet，测试一下。
可以访问127.0.0.1:27107.但是不能访问服务器ip。
用`docker exec`命令进入容器执行
```
#更新源
apt-get update
# 安装 vim
apt-get install vim
# 修改 mongo 配置文件
vim /etc/mongod.conf.orig

#配置用户密码
mongo
use <admin>

db.createUser({
    user: '<root>',
    pwd: '<root>',
    roles: [{ role: 'readWrite', db:'<admin>'}]
})
```
发现还是不能访问，直接关掉防火墙却可以。发现问题是在于防火墙。
之前使用的命令是
```
firewall-cmd --zone=public --add-port=27107/tcp --permanent
```

更换命令后即可在局域网访问
```
firewall-cmd --add-port=27017/tcp --permanent
success
firewall-cmd --query-port=27017/tcp --permanent
yes
```


