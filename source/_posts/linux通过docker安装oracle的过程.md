---
title: linux通过docker安装oracle的过程
date: 2020-04-01 15:17:30
tags: oracle
---

# linux通过docker安装oracle的过程
> 业务需要，要在linux服务器上装一个oracle。作为一个没有安装经验的我来说，首先想到的就是整一个docker镜像，run一下岂不是很快乐。但是oracle公司早就防了一手，也才有的这个记录过程。
> 如果仅仅是个人使用，第一次的尝试就可以安装好oracle数据库。但是里面的数据太杂乱，就有了后面的尝试。

<!--more-->

## 第一次尝试
首先熟练的打开百度，打算去找一个镜像文档。排位最靠前的是`sath89/oracle`这个镜像的使用博客。但是可惜的是直接用`docker search oracle`命令已经搜不到这个镜像了。

改用第二个镜像`registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g`

参考[https://blog.csdn.net/qq_27050005/article/details/81479171](https://blog.csdn.net/qq_27050005/article/details/81479171) 这篇博客来进行设置。
```
#pull镜像
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
#数据持久化：
docker run -d -p 49160:22 -p 49161:1521 -v /jie:/u01/app/oracle/ --name xe  registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
 
#====数据库默认信息
#hostname: localhost
#port: 1521
#sid: helowin
#username: system
#password: helowin
 
docker exec -it xe bash
su root # 密码：  helowin
#====vi /etc/profile 并在文件最后添加如下命令
#begin
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH
# esc :wq

#这条保存配置命令如果不配合，在~/.bashrc里面加一句source /etc/profile的话。再次进入后会失效
source /etc/profile

# 加个软连接
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
 
#====登录oracle数据库 ，修改密码
su oracle ;
sqlplus  
# username: system
# password: helowin
alter user system identified by 123456;
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
#退出
#===使用数据库工具 登录oracle
hostname: localhost
#记得开放你云服务的tcp 46161端口
port: 46161
sid: helowin
username: system
password: 123456

```

但是碰到了容器打开后无法通过服务器ip+49161访问数据库，linux本机用`wget`命令也无法联通。

查看防火墙端口
```
开启端口
[root@centos7 ~]# firewall-cmd --zone=public --add-port=1521/tcp --permanent

重启防火墙：

[root@centos7 ~]# firewall-cmd --reload

查询端口号1521 是否开启：
[root@centos7 ~]# firewall-cmd --query-port=1521/tcp

查询有哪些端口是开启的:

[root@centos7 ~]# firewall-cmd --list-port

--zone #作用域
--add-port=1521/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效
```

而用腾讯云的机器同样配置却可以访问。

尝试更改网络映射失败后，更换了参数，使用--net=host
```
docker run -d  --net=host -v /root/oracle:/u01/app/oracle/ --name oracle11g  registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```
> 此处命令如果没有删除端口映射的话，会报一个warning，提示端口映射无效。

这样配置了之后可以成功的在其他局域网机器上访问。

但是问题来了，这个库里自己存在了很多表。大佬说找找有没有纯净一点的。

## 第二次
又找到一个博客[https://mileslin.github.io/2019/06/%E4%BD%BF%E7%94%A8-Docker-%E5%BB%BA%E7%AB%8B-Oracle-%E6%9C%8D%E5%8B%99/](https://mileslin.github.io/2019/06/%E4%BD%BF%E7%94%A8-Docker-%E5%BB%BA%E7%AB%8B-Oracle-%E6%9C%8D%E5%8B%99/) ，这次用的是dockerhub上的oracle提供的镜像。（夸赞一下某个不存在的搜索引擎）

打开[Oracle Database Enterprise Edition](https://hub.docker.com/_/oracle-database-enterprise-edition)，需要点击`Proceed to Checkout` 。因为Oracle将镜像放在了[Docker Store](https://success.docker.com/article/store) （Docker提供给厂商进行商业行为的Repository）
![Proceed to Checkout](b1.png)

这样的话就需要我们登录dockerhub，点击之后需要填写一下资料
![Agreement](b2.png)

接下来到Docker Profile的My Content就可以找到Oracle的内容
![Added_to_my_content](b3.png)

点击`Setup`，就可以看到Oracle镜像的pull地址。

问题在于有点卡，下载了8个多小时。这中间还中断了几次，我就去尝试了另外的镜像。

下载完后，按照官方文档去启动容器.因为之前的端口局域网其他机器不能访问的问题没有解决，所以这里还是使用host模式。
```
docker run -d -it --name oracle -v /home/OracleDBData:/ORCL --net=host store/oracle/database-enterprise:12.2.0.1
```
成功启动后，进入容器内部
```
docker exec -it 4d7f(容器id简写) bash

```

测试用户名密码，提示要用sysdba角色登录。于是边用下面的命令登录成功。
```
sqlplus / as sysdba

username:system
password:Oradoc_db1

#sqlplus system/Oradoc_db1 也行

```

但是用navicat去连的时候，却报
```
ORA-12505, TNS:listener does not currently know of SID given in connect descriptor
```

第一个反应是博客记录有点问题，去找了找官方的评论区。有推荐用`ORCLPDB1.localdomain`做服务名，也是不行。再看看官方文档，和博客的说法一样。
```
DB_SID
This parameter changes the ORACLE_SID of the database. The default value is set to ORCLCDB
```

找了[stackoverflow.com](https://stackoverflow.com/questions/18192521/ora-12505-tnslistener-does-not-currently-know-of-sid-given-in-connect-descript/46848944) 上的一篇博客。测试了一下进入sqlplus后执行`ALTER SYSTEM REGISTER`命令。还是不行

大佬提供了一篇博客[https://blog.csdn.net/shen_xiao_wei/article/details/5729463](https://blog.csdn.net/shen_xiao_wei/article/details/5729463)
通过博客里的`find -name listener.ora`命令，找到文件
```
vi ./u01/app/oracle/product/12.2.0/dbhome_1/network/admin/samples/listener.ora
```
发现里面全都是被注释的语句，自己加上以下的语句并没有什么变化，依旧无法连接。
```

SID_LIST_LISTENER =
        (SID_LIST =
             (SID_DESC =
                   (SID_NAME = ORCLCDB)
                   (ORACLE_HOME = /private/app/oracle/product/8.0.3)
                   (PROGRAM = ORCLCDB)
             )
      
        )
```
再看一看另外一个同名文件
```
vi /u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB/listener.ora
```
里面只有以下的语句，把上面的语句加入后反而`lsnrctl`无法启动了。
```
LISTENER =   (DESCRIPTION_LIST =     (DESCRIPTION =       (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))       (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))     )   )

#报错
TNS-01150: The address of the specified listener name is incorrect
```

再次仔细阅读了**stackoverflow.com**的博客，发现评论里有一条说法
```
Now you can see the service
Even if don't see try this one out

sqlplus /nolog  
conn system  
alter system set local_listener = '(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521))' scope = both;  
alter system register;  
exit  
lsnrctl status

```
尝试了一下，效果十分明显。错误已经变成了
```
ORA-01017： invalid username/password;logon denied
```
在navicat里面重新输入了一遍SID、账号、密码和角色。点击测试连接，显示成功。点击确认，显示错误。
![](b4.png)
忽略中间大起大落的心态问题，去百度了一下。简单的在navicat中删除了之前第一次成功时建立的服务器连接解决问题。


## 第三次
大佬提供了另外的一个博客[使用Docker安装oracle 11g](https://www.35youth.cn/685.html) 。
这篇使用的镜像是jaspeen/oracle-11g。之前曾经直接下载过，但是run后直接闪退，就放弃了。这次按照博客的步骤来
```
docker pull jaspeen/oracle-11g
```

准备oracle压缩包，在[Oracle官网](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html) 分别下载 linux.x64_11gR2_database_1of2.zip 和 linux.x64_11gR2_database_2of2.zip两个压缩包，下载完成后解压到home目录(如下目录结构)
```
  home
    └─database
        ├─doc
        ├─install
        ├─response
        ├─rpm
        ├─sshsetup
        ├─stage
        ├─runInstaller
        └─welcome.html
```

略过为什么要解压在这个包下，我直接启动容器

```
docker run --privileged -d -v /home/oracle:/install/database --name jaspeen --net=host jaspeen/oracle-11g
```
这次的命令是我输入错了。我的压缩包解压在home目录，我指定/home/oracle后导致启动报错
**/install/database/runInstaller: No such file or directory**。

再次启动还是不行
```
docker run --privileged --name jaspeen --net=host -v /home:/install jaspeen/oracle-11g
```

报错
```
Database is not installed. Installing...
Installing Oracle Database 11g
bash: /install/database/runInstaller: Permission denied
```
不知道是`--privileged`没有生效，还是权限不足。尝试Selinux临时关闭，`setenforce 0`也不能解决问题。

## 第四次
又换了一个镜像，这次我看到镜像描述里说是sath89/oracle-12c的复制。就尝试一下。
```
docker pull truevoly/oracle-12c
docker run -d --privileged --net=host -v /home/oracle:/u01/app/oracle/ --name sath89  truevoly/oracle-12c
```

结果启动容器失败
```
Database not initialized. Initializing database.
Starting tnslsnr
Cannot create directory "/u01/app/oracle/cfgtoollogs/dbca".

Unique database identifier check passed.
Error writing into silent log -- /u01/app/oracle/cfgtoollogs/dbca/silent.log_2020-04-01_07-33-18-AM (No such file or directory)

/u01/app/oracle/ has enough space. Required space is 6140 MB , available space is 40222 MB.
File Validations Successful.
Error writing into silent log -- /u01/app/oracle/cfgtoollogs/dbca/silent.log_2020-04-01_07-33-18-AM (No such file or directory)
Cannot create directory "/u01/app/oracle/cfgtoollogs/dbca/xe".
Error writing into silent log -- /u01/app/oracle/cfgtoollogs/dbca/silent.log_2020-04-01_07-33-18-AM (No such file or directory)
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/silent.log_2020-04-01_07-33-18-AM" for further details.
Error in file copy from </u01/app/oracle/cfgtoollogs/dbca/silent.log_2020-04-01_07-33-18-AM> to </u01/app/oracle/cfgtoollogs/dbca/xe.log>
```
再次选择放弃。返回第二个官方镜像
