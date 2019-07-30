---
title: 在windows7上安装docker和hbase镜像和hadoop镜像
date: 2019-06-21 21:43:56
tags: docker
---

# 在windows7上安装docker和hbase镜像和hadoop镜像

> 在之前的本机安装Hbase环境没有进展后，更换了docker镜像来搭建测试环境。~~参考[基于 Docker 搭建 Mac 本地 HBase 环境](https://www.colabug.com/4772817.html)~~（原文网址已经404）

<!--more-->

## 安装 Docker
本地环境为windows7 64位系统，直接从官网[https://docs.docker.com/toolbox/toolbox_install_windows/](https://docs.docker.com/toolbox/toolbox_install_windows/) 上面安装**Docker Toolbox**。按照指示一直下一步即可完成安装。
### 启动 Docker
- 第一次启动会默认安装一个default命名的虚拟机。因为需要阿里的镜像加速的缘故，我用
```
docker-machine stop default
docker-machine rm default
```
这两个命令来删除了默认虚拟机。之后再根据阿里的镜像加速配置来操作

- 创建一台安装有Docker环境的Linux虚拟机，指定机器名称为default，同时配置Docker加速器地址。
```
docker-machine create --engine-registry-mirror=https://ms4x763y.mirror.aliyuncs.com -d virtualbox default
```
- 查看机器的环境配置，并配置到本地，并通过Docker客户端访问Docker服务。
```
docker-machine env default
eval "$(docker-machine env default)"
docker info
```
- 因为我本地环境为windows7并且没有装Docker for Windows，所以忽略官网文档其他的操作配置。

## 安装 Hbase
- 获取容器镜像
```
docker pull harisekhon/hbase
```
- 启动容器
```
docker run -d -h myhbase -p 2181:2181 -p 8080:8080 -p 8085:8085 -p 9090:9090 -p 9095:9095 -p 16000:16000 -p 16010:16010 -p 16020:16020 -p 16030:16030 --name hbase1.3.1 harisekhon/hbase
```
> - -d: 后台启动。
> - -h: 容器主机名，必须设置该项并配置 hosts，否则无法连通容器。
> - -p: 网络端口映射，这里只把要使用的端口（zookeeper端口、HBase Master端口、HBase RegionServer端口等）映射了出来，你可以根据自己需要进行端口映射。
> - --name: 容器别名。

添加说明：大多数的博客贴在进行到这里的时候都是映射端口为16201和16301，没有16020和16030。经过我查看http://192.168.99.100:16010/master-status 页面发现我安装的最新hbase2.1.1版本默认启动的是16020和16030。这里特别注意

### 进入容器
- 进入hbase命令行
```
docker exec -it hbase1.3.1 /bin/bash
/hbase/bin/hbase shell
hbase(main):001:0> create 't1', {NAME => 'f1', VERSIONS => 1}
```
> 在执行完第一个命令进入hbase容器后，可以执行**netstat -anp | grep 16000** 来查看hbase绑定的ip 172.17.0.2

- 运行结果
```
Created table t1
Took 2.6696 seconds
=> Hbase::Table - t1
```

这样就完成了整个安装过程。

## 安装hadoop（没有必要，上面的已经集成了）
- 搜索hadoop镜像
```
docker search hadoop
```

- 选择第一个镜像
```
docker pull sequenceiq/hadoop-docker
```

当命令行显示**Status: Downloaded newer image for sequenceiq/hadoop-docker:latest**时候开始下一步。

- 启动镜像
```
docker run -it --name hadoop sequenceiq/hadoop-docker /etc/bootstrap.sh -bash
```

- 测试运行:
```
cd $HADOOP_PREFIX
# run the mapreduce
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep input output 'dfs[a-z.]+'

# check the output
bin/hdfs dfs -cat output/*
```

## 删除容器
使用以下命令删除单个或多个容器。
```
docker rm <CONTAINER ID|NAME> <CONTAINER ID|NAME>
```