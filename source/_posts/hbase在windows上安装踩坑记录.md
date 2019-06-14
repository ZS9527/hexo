---
title: hbase在windows上安装踩坑记录
date: 2019-06-08 22:39:19
tags: hbase
---
# hbase在windows上安装踩坑记录
> 记录一下在windows7机器上安装hbase时遇到的问题，以便于以后使用。

<!--more-->

## 前置条件：
已经安装好了hadoop-3.1.1，并且从[apache hbase-2.1.5下载地址](https://www.apache.org/dyn/closer.lua/hbase/2.1.5/hbase-2.1.5-bin.tar.gz)中下载了压缩包。
## 修改配置文件/conf/hbase-site.xml：
```
<configuration>
	<property>  
		<name>hbase.rootdir</name>  
		<value>file:///D:/hadoop/hbase-2.1.5/tmp/hbase/root</value>  
	</property>  
	<property>  
		<name>hbase.tmp.dir</name>  
		<value>D:/hadoop/hbase-2.1.5/tmp/hbase/tmp</value>  
	</property>  
	<property>  
		<name>hbase.zookeeper.quorum</name>  
		<value>127.0.0.1</value>  
	</property>  
	<property>  
		<name>hbase.zookeeper.property.dataDir</name>  
		<value>D:/hadoop/hbase-2.1.5/tmp/hbase/zoo</value>  
	</property>  
	<property>
		<name>hbase.cluster.distributed</name>
		<value>false</value>
	</property>
	<property>    
        <name>hbase.zookeeper.property.clientPort</name>    
        <value>2181</value>    
    </property> 
</configuration>
```
其中，**hbase.cluster.distributed**为HBase以分布式模式进行，这个功能在win下不支持。如果在win中设置为true的话。在运行start-hbase.cmd启动时，会报错**This is not implemented yet. Stay tuned**。

但是设置为false的话，会启动hbase自带的zookeeper，需要把本地装的zookeeper关闭。

## 修改hbase-env.cmd：
```
set JAVA_HOME=G:\JDK8_64
set HBASE_MANAGES_ZK=false
```

## 环境变量的设置：
添加系统变量 HBASE_HOME : D:/hadoop/hbase-2.1.5

在Path中添加%HBASE_HOME%\bin

## 启动HBase服务:（未完成）
在启动HBase之前，我们需要先启动Hadoop；

接着，在\hbase\bin内打开命令行。运行**start-hbase.cmd**，即可完成启动