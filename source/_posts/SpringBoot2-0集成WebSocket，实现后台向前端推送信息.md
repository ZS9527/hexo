---
title: SpringBoot2.0集成WebSocket，实现推送信息
date: 2019-04-02 21:10:03
tags: webSocket
---
# SpringBoot2.0集成WebSocket，实现推送信息

> 原博客为[https://blog.csdn.net/moshowgame/article/details/80275084](https://blog.csdn.net/moshowgame/article/details/80275084)。这里是根据原博客搭建成功之后的记录，以后使用时可以直接套用。

<!--more-->

## 什么是WebSocket?
WebSocket是HTML5下一种新的协议。它实现了浏览器与服务器全双工通信——允许服务器主动发送信息给客户端，能更好的节省服务器资源和带宽并达到实时通讯的目的。它与HTTP一样通过已建立的TCP连接来传输数据，但是它和HTTP最大不同是：
- WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样；
- WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。


![](b1.jpg)	

## 为什么需要 WebSocket？
相比HTTP长连接，WebSocket有以下特点：
- 是真正的全双工方式，建立连接后客户端与服务器端是完全平等的，可以互相主动请求。而HTTP长连接基于HTTP，是传统的客户端对服务器发起请求的模式。
- HTTP长连接中，每次数据交换除了真正的数据部分外，服务器和客户端还要大量交换HTTP header，信息交换效率很低。Websocket协议通过第一个request建立了TCP连接之后，之后交换的数据都不需要发送 HTTP header就能交换数据，这显然和原有的HTTP协议有区别所以它需要对服务器和客户端都进行升级才能实现（主流浏览器都已支持HTML5）。此外还有 multiplexing、不同的URL可以复用同一个WebSocket连接等功能。这些都是HTTP长连接不能做到的。

## spring boot 如何使用WebSocket

### 引入jar包

使用springboot的内置tomcat时，就不需要引入javaee-api了，spring-boot已经包含了。使用springboot的websocket功能首先引入springboot组件。
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
            <version>1.3.5.RELEASE</version>
</dependency>
```
顺便说一句，springboot的高级组件会自动引用基础的组件，像spring-boot-starter-websocket就引入了spring-boot-starter-web和spring-boot-starter，所以不要重复引入。

### 配置WebSocketConfig

启用WebSocket的支持也是很简单，几句代码搞定
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class WebSocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```

### 实现WebSocketServer

因为WebSocket是类似客户端服务端的形式(采用ws协议)，那么这里的WebSocketServer其实就相当于一个ws协议的Controller
直接@ServerEndpoint("/websocket")@Component启用即可，然后在里面实现@OnOpen,@onClose,@onMessage等方法
```
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArraySet;

import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
import org.springframework.stereotype.Component;
import cn.hutool.log.Log;
import cn.hutool.log.LogFactory;
import lombok.extern.slf4j.Slf4j;


@ServerEndpoint("/websocket/{sid}")
@Component
public class WebSocketServer {
	
	static Log log=LogFactory.get(WebSocketServer.class);
    //静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
    private static int onlineCount = 0;
    //concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
    private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<WebSocketServer>();

    //与某个客户端的连接会话，需要通过它来给客户端发送数据
    private Session session;

    //接收sid
    private String sid="";
    /**
     * 连接建立成功调用的方法*/
    @OnOpen
    public void onOpen(Session session,@PathParam("sid") String sid) {
        this.session = session;
        //加入set中
        webSocketSet.add(this);
        //在线数加1
        addOnlineCount();
        log.info("有新窗口开始监听:"+sid+",当前在线人数为" + getOnlineCount());
        this.sid=sid;
        try {
        	 sendMessage("连接成功");
        } catch (IOException e) {
            log.error("websocket IO异常");
        }
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        //从set中删除
        webSocketSet.remove(this); 
         //在线数减1
        subOnlineCount();
        log.info("有一连接关闭！当前在线人数为" + getOnlineCount());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息*/
    @OnMessage
    public void onMessage(String message, Session session) {
    	log.info("收到来自窗口"+sid+"的信息:"+message);
        //群发消息
        for (WebSocketServer item : webSocketSet) {
            try {
                item.sendMessage(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

	/**
	 * 
	 * @param session
	 * @param error
	 */
    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }
	/**
	 * 实现服务器主动推送
	 */
    public void sendMessage(String message) throws IOException {
        this.session.getBasicRemote().sendText(message);
    }


    /**
     * 群发自定义消息
     * */
    public static void sendInfo(String message,@PathParam("sid") String sid) throws IOException {
    	log.info("推送消息到窗口"+sid+"，推送内容:"+message);
        for (WebSocketServer item : webSocketSet) {
            try {
            	//这里可以设定只推送给这个sid的，为null则全部推送
            	if(sid==null) {
            		item.sendMessage(message);
            	}else if(item.sid.equals(sid)){
            		item.sendMessage(message);
            	}
            } catch (IOException e) {
                continue;
            }
        }
    }
/**
  * 获得计数器数量
  * */
    public static synchronized int getOnlineCount() {
        return onlineCount;
    }
/**
  * 增加计数器数量
  * */
    public static synchronized void addOnlineCount() {
        WebSocketServer.onlineCount++;
    }
/**
  * 删减计数器数量
  * */
    public static synchronized void subOnlineCount() {
        WebSocketServer.onlineCount--;
    }
}
```

### 消息推送 （并没有经过实际测试）
推送新信息，可以再自己的Controller写个方法调用WebSocketServer.sendInfo();即可

```
@Controller
@RequestMapping("/checkcenter")
public class CheckCenterController {

	//页面请求
	@GetMapping("/socket/{cid}")
	public ModelAndView socket(@PathVariable String cid) {
		ModelAndView mav=new ModelAndView("/socket");
		mav.addObject("cid", cid);
		return mav;
	}
	//推送数据接口
	@ResponseBody
	@RequestMapping("/socket/push/{cid}")
	public ApiReturnObject pushToWeb(@PathVariable String cid,String message) {  
		try {
			WebSocketServer.sendInfo(message,cid);
		} catch (IOException e) {
			e.printStackTrace();
			return ApiReturnUtil.error(cid+"#"+e.getMessage());
		}  
		return ApiReturnUtil.success(cid);
	} 
} 
	
```

### 页面发起socket请求
然后在页面用js代码调用socket，当然，太古老的浏览器是不行的，一般新的浏览器或者谷歌浏览器是没问题的。还有一点，记得协议是ws的哦，如果像我这样封装了一些basePath的路径类，可以replace(“http”,“ws”)来替换协议

```
    <script> 
    var socket;  
    if(typeof(WebSocket) == "undefined") {  
        console.log("您的浏览器不支持WebSocket");  
    }else{  
        console.log("您的浏览器支持WebSocket");  
        	//实现化WebSocket对象，指定要连接的服务器地址与端口  建立连接  
            //等同于socket = new WebSocket("ws://localhost:8083/checkcentersys/websocket/20");  
            socket = new WebSocket("${basePath}websocket/${cid}".replace("http","ws"));  
            //打开事件  
            socket.onopen = function() {  
                console.log("Socket 已打开");  
                //socket.send("这是来自客户端的消息" + location.href + new Date());  
            };  
            //获得消息事件  
            socket.onmessage = function(msg) {  
                console.log(msg.data);  
                //发现消息进入    开始处理前端触发逻辑
            };  
            //关闭事件  
            socket.onclose = function() {  
                console.log("Socket已关闭");  
            };  
            //发生了错误事件  
            socket.onerror = function() {  
                alert("Socket发生了错误");  
                //此时可以尝试刷新页面
            }  
            //离开页面时，关闭socket
            //jquery1.8中已经被废弃，3.0中已经移除
            // $(window).unload(function(){  
            //     socket.close();  
            //});  
    }
    </script> 

```

## 运行效果

![](b2.jpg)	

![](b3.jpg)	