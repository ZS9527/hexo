---
title: 日常业务和hadoop初学
date: 2019-06-01 15:32:35
tags: hadoop
---

# 日常业务和hadoop初学

> 记录上周遇到的一些业务问题，和在学习hadoop安装中碰到的bug。

<!--more-->

## 日常遇到的代码问题
1. 使用idea进行gradle build项目时候遇到警告
警告: Generating equals/hashCode implementation but without a call to superclass, even though this class does not extend java.lang.Object. If this is intentional, add '@EqualsAndHashCode(callSuper=false)' to your type.
在报警告的类里加入'@EqualsAndHashCode(callSuper=false)注解就可以了。
2. settings.gradle文件中可以修改打包后的jar包文件名字，记得打包前修改一下。

## hadoop初学（windows764系统版本）

### 安装过程
参考csdn上的[Hadoop2.7.6在Windows7单机部署](https://blog.csdn.net/chy2z/article/details/80484848)，此文的安装过程。其中遇到了一些问题，记录如下：

### 错误汇总：

#### 使用start-all.cmd命令后产生的错误
1.  INFO datanode.DataNode: Shutdown complete.
ERROR datanode.DataNode: Exception in secureMain
java.io.IOException: **No services to connect, missing NameNode address.**
at org.apache.hadoop.hdfs.server.datanode.BlockPoolManager.refreshNamenodes(BlockPoolManager.java:165)

2.  INFO namenode.NameNodeUtils: fs.defaultFS is file:///
 ERROR **namenode.NameNode: Failed to start namenode.**
java.lang.IllegalArgumentException: Invalid URI for NameNode address (check fs.defaultFS): file:/// has no authority.
at org.apache.hadoop.hdfs.DFSUtilClient.getNNAddress(DFSUtilClient.java:697)

3. error **Couldn't find a package.json file in "D:\\hadoop\\hadoop-3.1.1\\sbin"**
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.

4. Error: **JAVA_HOME is incorrectly set.**

#### 解决方法
第一和第二都是一个原因导致的，是我在配置文件的时候遗漏了core-site.xml这个配置文件所导致的。复制网上的配置后即可解决

第三个错误是因为我将前端的包管理工具yarn配置到了环境变量里。当使用start-all.cmd命令时，其中的yarn被系统判断为前端的yarn导致。删除环境变量中的path变量中的yarn路径，再重启电脑就可以解决这个问题。

> 大多数的jdk报错原因都是因为jdk的路径中有空格，或者是hadoop-env.cmd配置文件里的JAVA_HOME=%JAVA_HOME%导致的。

第四个错误从网上找到两种解决方案：
1.用路径替代符C:\PROGRA~1\Java\jdk1.8.0_91
PROGRA~1  ===== C:\Program Files 目录的dos文件名模式下的缩写
长于8个字符的文件名和文件夹名，都被简化成前面6个有效字符，后面~1，有重名的就 ~2,~3,
2.用引号括起来"C:\Program Files"\Java\jdk1.8.0_91。
第一个方法经过尝试后依旧不能解决路径空格的问题，第二个可以解决