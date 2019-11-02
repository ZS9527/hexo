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

### 删除服务
```
sc delete MySQL3（服务名称）
```
为了防止出现一不小心把命令行关闭了，而且还没记录密码，需要重新开始的情况。在此记录一下删除命令
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

## 后记：
在一次安装时，新建完my.ini文件执行时。出现了一下的这个错误：

```
error: Found option without preceding group in config file: D:\mysql-5.6.24-win32\my.ini at line: 1  
Fatal error in defaults handling. Program aborted
```

查询后发现是文件的编码问题，从utf-8改为ANSI后就可以了。

## 9月更新
发现这个博客应该配合**配置mysql远程访问**一起使用会比较好。加入后续步骤

### 更改root用户访问ip限制
首先要进入你想要更改的mysql：
```sql
mysql -u root -p --protocol=tcp --host=localhost --port=3308（选择端口）
```

然后查询和修改
```sql
use mysql;
```
```sql
select host, user, authentication_string, plugin from user; 
```
可以看到现在的root用户的host是localhost
```sql
update user set host = '%' where user = 'root';
```
修改后需要刷新一下权限
```sql
flush privileges;
```
然后开启全部访问权限
```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```
再次刷新
```sql
flush privileges;
```

完成以上步骤后就可以从本机中测试连接服务器的mysql了。

如果还是连接失败的话，记得开启服务器防火墙上的入站规则中的端口3306和3308（我自己配置的mysql端口就是这两个）