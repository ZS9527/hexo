---
title: RabbitMQ中的stomp初步部署
date: 2019-10-19 16:47:45
tags: rabbitmq
---

# RabbitMQ中的stomp初步部署

> 本文是为了防止我使用的参考网页404，而做的备份。原网页地址为：[https://blog.csdn.net/weixin_40461281/article/details/81806921](https://blog.csdn.net/weixin_40461281/article/details/81806921)

<!--more-->

## 非docker安装RabbitMQ的stomp部署
1. 进入到RabbitMQ安装目录下的sbin文件夹内。
2. Shift加右键进入命令行
3. 执行命令 
```
rabbitmq-plugins enable rabbitmq_web_stomp
```
```
rabbitmq-plugins enable rabbitmq_web_stomp_examples
```
4. 成功（没有报错消息）后重启RabbitMQ
5. 尝试使用stomp.js进行连接
6. 连接成功
## docker安装RabbitMQ的stomp部署
1. 执行docker命令，容器id可以通过docker ps查询
```
docker exec -it {容器id} /bin/sh -c "[ -e /bin/bash ] && /bin/bash || /bin/sh"
```
2. 进入容器后的命令和上面的非docker一致
3. 开启后需要修改容器的开放端口。我个人为了省事，直接删除容器后新建容器时就开放了15674、61613。
4. 停止容器
```
docker stop {容器名}
```
5. 删除容器
```
docker rm {容器名}
```