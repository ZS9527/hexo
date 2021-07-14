---
title: Linux后台启动程序操作
date: 2020-10-31 18:31:09
tags: Linux
---

# Linux后台启动程序操作
> 感觉越来越难写了。有点累，好想放个寒假。
> 本文主要用于记录我操作 Linux 的过程。包含后台启动程序、找到占用的端口、关闭占用端口程序。
> 
<!--more-->

## 后台启动程序
来自 [runnoob.com](https://www.runoob.com/linux/linux-comm-nohup.html)
**语法格式：**
```
nohup Command [ Arg … ] [　& ]
```

**参数说明：**
Command：要执行的命令。

Arg：一些参数，可以指定输出文件。

&：让命令在后台执行，终端退出后命令仍旧执行。

**实例**
以下命令在后台执行 root 目录下的 runoob.sh 脚本：
```
nohup /root/runoob.sh &
```
以下命令在后台执行 root 目录下的 runoob.sh 脚本，并重定向输入到 runoob.log 文件：
```
nohup /root/runoob.sh > runoob.log 2>&1 &
```
2>&1 解释：

将标准错误 2 重定向到标准输出 &1 ，标准输出 &1 再被重定向输入到 runoob.log 文件中。

- 0 – stdin (standard input，标准输入)
- 1 – stdout (standard output，标准输出)
- 2 – stderr (standard error，标准错误输出)

**本人使用：**
```
nohup java -jar v.jar > v.log 2>&1 &
```
**查看后台进程**
```
ps -aux|grep java
```
## 找到占用的端口
来自[https://www.cnblogs.com/zjfjava/p/10513399.html](https://www.cnblogs.com/zjfjava/p/10513399.html)
用于查看指定端口号的进程情况
**语法格式：**
```
netstat -tunlp | grep 端口号
```
**参数说明：**
几个参数含义

- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名

**本人使用**
```
netstat -tunlp | grep 8080
```

## 关闭占用端口程序
**本人使用**
```
kill -9  进程号PID
```