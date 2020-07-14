---
title: linux搭建ftp
date: 2020-02-15 16:35:38
tags: linux
---

# linux搭建ftp
> 记录安装ftp过程以及遇到的问题。
<!--more-->

## 安装流程
1. 首先要确认服务器的端口开放情况，ftp被动模式使用21端口。

2. 安装vsftp软件包
```
yum install -y vsftpd
```

3. 修改ftp配置文件，默认位置在/etc/vsftpd/。最好备份一下原始文件以便后期调整。

4. ftp配置文件vsftpd.conf详细内容
```
anonymous_enable=NO                    #不允许匿名用户登陆 

local_enable=YES                      #vsftpd所在系统的用户可以登录vsftpd 

write_enable=YES                      #允许使用任何可以修改文件系统的FTP的指令 

local_umask=000                        #匿名用户新增文件的umask数值 

anon_upload_enable=NO                  #匿名用户不可以上传文件 

anon_mkdir_write_enable=NO            #匿名用户不可以修改文件 

xferlog_enable=YES                    #启用一个日志文件，用于详细记录上传和下载。                
use_localtime=YES                      #使用本地时间而不是GMT 

vsftpd_log_file=/var/log/vsftpd.log    #vsftpd日志存放位置 

dual_log_enable=YES                    #用户登陆日志 

connect_from_port_20=YES              #开启20端口     

xferlog_file=/var/log/xferlog          #记录上传下载文件的日志 

xferlog_std_format=YES                #记录日志使用标准格式 

idle_session_timeout=600              #登陆之后超时时间60秒，登陆之后，一分钟不操作，就会断开连接。 

chroot_local_user=YES                  #用于指定用户列表文件中的用户,是否允许切换到上级目录      

listen=YES                            #开启监听 

pam_service_name=vsftpd            #验证文件的名字 

tcp_wrappers=YES                      #支持tcp_wrappers,限制访问

(/etc/hosts.allow,/etc/hosts.deny) 

pasv_min_port=35000  
pasv_max_port=45000 
pasv_enable=YES 
pasv_promiscuous=YES 

anon_other_write_enable=YES

userlist_enable=YES                    
userlist_deny=NO                      #允许由userlist_file指定文件中的用户登录FTP服务器
```
这里说明一下用户权限问题，和配置文件user_list与ftpusers有关。
1:userlist_enable和userlist_deny两个选项联合起来针对的是：本地全体用户（除去ftpusers中的用户）和出现在user_list文件中的用户以及不在在user_list文件中的用户这三类用户集合进行的设置。
2:当且仅当userlist_enable=YES时：userlist_deny项的配置才有效，user_list文件才会被使用；当其为NO时，无论userlist_deny项为何值都是无效的，本地全体用户（ftpusers中的用户不能登入FTP）都可以登入FTP。
3:当userlist_enable=YES时，userlist_deny=YES时：user_list是一个黑名单，即：所有出现在名单中的用户都会被拒绝登入。
4:当userlist_enable=YES时，userlist_deny=NO时：user_list是一个白名单，即：只有出现在名单中的用户才会被准许登入(user_list之外的用户都被拒绝登入)。另外需要特别提醒的是：使用白名单后，匿名用户将无法登入！除非显式在user_list中加入一行：anonymous。

5. 重启vsftpd
```
# 重启
service vsftpd restart
# 开启
service vsftpd start
# 停止
service vsftpd stop
```

6. 添加ftp用户
useradd 用户名 -s /sbin/nologin //限定用户test不能telnet，只能ftp。

usermod -s /sbin/bash 用户名 //用户恢复正常 。该账户路径默认指向/home/ftpadmin目录。

设置ftpadmin用户密码，运行命令：”passwd ftpadmin” 。 输入两次密码，匹配成功后，就设置好了ftpadmin用户的密码了。

测试连接，您可以在“我的电脑”地址栏中输入 ftp://IP 来连接FTP服务器，根据提示输入账户密码。

到这里正常的ftp配置已经完成了。

7. 删除用户账号
使用 userdel -r xiaoluo命令删除。这个命令会删除用户的文件夹。
## 遇到的问题（已解决）
我想要修改一下ftp用户的默认文件夹。当用户登录ftp时，本身的默认文件夹位置是在/home/用户名。
查询后使用下面的命令更换
```
usermod -d /tmp test (test为你的用户名)

```
更换后发现并不能从浏览器登录了。
在linux中使用命令登录：
```
ftp 127.0.0.1
```
出现如下的错误提示：-bash: ftp: command not found。
使用命令，安装ftp客户端。
```
yum -y install ftp
```
再次登录，输入用户名和密码后。报500 OOPS: cannot change directory。
查询后尝试使用下面的命令关闭了Selinux
```
# 查询Selinux状态
sestatus -v

```
修改/etc/selinux/config 文件。将SELINUX=enforcing改为SELINUX=disabled。依旧不行。
再次尝试修改文件夹权限
```
chown 用户组:用户名 -R /tmp 
chmod -R 777 /tmp
```
还是不行。查询后大多数都是用这几种有关的命令解决。目前没有找到原因，暂时使用默认文件夹


### 解决方式

> 2020年03月13号更新

呼叫大佬后，大佬也没明白我都做了什么操作。发给了我一个博客，照着一步一步操作后很顺利的成功了。

把博客贴在这里[https://blog.csdn.net/wsyh12345678/article/details/83244940](https://blog.csdn.net/wsyh12345678/article/details/83244940)

和我之前的配置文件有所不同的地方
```
dirmessage_enable=YES

xferlog_enable=YES

ascii_upload_enable=YES

ascii_download_enable=YES
```
同样将**chroot_local_user=YES**设置为yes。

被列入此文件的用户，在登录后将不能切换到自己目录以外的其他目录

从而有利于FTP服务器的安全管理和隐私保护。此文件需自己建立 ,里面存入用户名，一个用户占一行chroot_list_file=/etc/vsftpd/chroot_list

如果设置为
chroot_local_user＝YES#所有用户都只能访问自己的主目录

chroot_list_enable=YES(这行可以没有, 也可以有)

chroot_list_file=/etc/vsftpd.chroot_list

那么, 凡是加在文件vsftpd.chroot_list中的用户都是不受限止的用户，即, 可以浏览其主目录的上级目录.

如果不希望某用户能够浏览其主目录上级目录中的内容,可以如上设置,然后在文件vsftpd.chroot_list中去掉或不添加该用户即可。

也可以如下配置

chroot_local_user＝NO

chroot_list_enable=YES(这行必须要有, 否则文件vsftpd.chroot_list不会起作用)

chroot_list_file=/etc/vsftpd.chroot_list

然后把所有不希望有这种浏览其主目录之上的各目录权限的用户添加到文件vsftpd.chroot_list中即可(一行一个用户名，此时, 在该文件中的用户都是不可以浏览其主目录之外的目录的)
**关键步骤：**

- Mkdir /data/wwwroot #这里是根据自己需求来建立（就是FTP用户可以访问那个目录）

- 进入用户配置目录，没有的话自己建立，注意和vsftpd.conf文件中的对应 user_config_dir=/etc/vsftpd/userconf
cd /etc/vsftpd/userconf

- 建立一个同登录用户名称一样的文件(在配置userconf文件夹里创建)
vim ftpuser

- 写入用户登录FTP后能访问的目录位置：

local_root=/www/test

- 重启VSftp：

/etc/init.d/vsftpd restart

之后即可用新建的用户访问自定义的文件夹了。