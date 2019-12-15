---
title: rabbitMQ远程连接失败问题
date: 2019-10-30 16:53:39
tags: rabbitmq
---
# rabbitMQ远程连接失败问题
> 将数据部署到服务器，程序开在本地时出现的远程连接问题。

<!--more-->

## 问题一
将rabbitMQ部署在服务器后，同事想要从本地连接web网页查看一下队列情况，发现根本打不开网页。程序连接报错：**Login was refused using authentication mechanism PLAIN. For details see the broker logfile.**

首先检查了一下服务器的端口打开情况，防火墙已经通过了所有的端口。
百度后查看了一下guest用户的权限，确认是最高权限的试用账号以及可以全host登录。
再次百度查找资料后（不是我找的），找到一个文章说guest用户只能在localhost本地登录。尝试新建用户root并开启所有权限。可以用这个新的账号远程登录了。

## 问题二
但是在程序中连接仍旧报错：**Received ERROR {message=[Bad CONNECT], content-type=[text/plain], version=[1.0,1.1,1.2], 
content-length=[26]} session=_system_ text/plain payload=non-loopback access denied**

查询到需要添加参数
```
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

@Override
public void configureMessageBroker(MessageBrokerRegistry config) {
    config.enableStompBrokerRelay("/topic")
            .setRelayHost("192.168.1.8")
            .setRelayPort(61613)
            .setClientLogin("user")
            .setClientPasscode("user")
            .setSystemLogin("user")
            .setSystemPasscode("user");

    //config.enableSimpleBroker("/topic");
    config.setApplicationDestinationPrefixes("/app");
}

@Override
public void registerStompEndpoints(StompEndpointRegistry registry) {
    registry.addEndpoint("/gs-guide-websocket").setAllowedOrigins("*").withSockJS();   
}
}
```
其中的setClientLogin到setSystemPasscode就是新增加的参数，设置为新建的root用户即可正常连接了。