---
title: windows同时安装多个mysql8.0服务
date: 2019-03-12 14:44:51
tags: sql
---

# windows同时安装多个mysql8.0服务

> 原作者地址为https://blog.csdn.net/m0_37890289/article/details/80003994。 我在mysql主从复制时使用的这篇文章，这里做一个备份。

<!--more-->

## 官网下载 mysql文件。官网下载链接：[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)
![](b1.png)
　　1. 选择mysql下载的系统版本。
　　2. 此处可以下载MSI安装包，图简单的朋友可以下载，然后“下一步”安装即可。
　　3. 此处下载ZIP压缩包版（这次记录ZIP压缩包安装方法）
## 解压下载好的ZIP文件，到自己喜欢的位置。
![](b2.png)
　　1. 因为我这次要安装多个mysql服务，为mysql的主从复制做准备，所以我的文件结构如上。
　　2. 官网下载的ZIP压缩包。
　　3. 解压3次副本。

## 在解压后的文件夹下新建my.ini文件。内容参考如下：

```
[mysql]
 
 
# 设置mysql客户端默认字符集
 
 
default-character-set=utf8 
 
 
[mysqld]
 
 
#设置3306端口
 
 
port = 3306 
 
 
# 设置mysql的安装目录
 
 
basedir=E:\mysql\mysql3\mysql-8.0.11-winx64
 
 
# 设置mysql数据库的数据的存放目录
 
 
datadir=E:\mysql\mysql3\mysql-8.0.11-winx64\data
 
 
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
 
 
character-set-server = utf8mb4
 
 
performance_schema_max_table_instances = 600
 
 
table_definition_cache = 400
 
 
table_open_cache = 256

```

处理完目录结构如下。
![](b3.png)
## 管理员身份运行cmd,并将切换到你解压过后的文件的bin目录下。
![](b4.png)
## 初始化mysql
```    
    mysqld --defaults-file=D:\mysql\mysql3\mysql-8.0.11-winx64\my.ini --initialize --console
```
注意路径的填写，要和自己的解压安装包路径一致。
执行完毕后，文件结构多了一个data目录。并记录初始化密码（后面第一次进入mysql修改密码需要）。
## 初始化mysql服务。
```
    mysqld install MySQL3 --defaults-file="D:\mysql\mysql3\mysql-8.0.11-winx64\my.ini"

```
![](b5.png)
## 启动mysql服务。 
```
    net start MySQL3
```
这里的服务名字和上面的命令中起的服务名一致
![](b6.png)
## 以root身份进入mysql。
```
    mysql -u root -p --protocol=tcp --host=localhost --port=3308（选择端口）
```
![](b7.png)
此处就需要上面初始化mysql时候系统生成的密码了。
![](b8.png)
## 修改root密码。
```
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'newpassword';
```
![](b9.png)
## 密码修改后刷新权限，就可以使用mysql了。
```
    flush privileges;
```
![](b10.png)

**这个方法可以在一台电脑上安装N个mysql服务，注意修改不同的端口号即可。**