
# springboot2.0开发websocket案例（含代码）

#### 1 概述  

##### 1.1 介绍
- WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工通信:允许服务器主动发送信息给客户端。 
- WebSocket协议跟http协议并没有太多的关系,不过使用了http的握手机制.
- http协议是应用很广的网络协议,它在通信前必须经过3次握手,它又分为短链接，长链接.短链接是每次请求都要三次握手才能发送自己的信息。即每一个request对应一个response。长链接是在一定的期限内保持链接。保持TCP连接不断开。客户端与服务器通信，必须要有客户端发起请求然后服务器返回结果。客户端是主动的，服务器是被动的。https是在http之上进行安全加密的协议,本质上还是http协议,通信方式还是一样.
- http协议在通信时,需要不停的进行连接,关闭,这是十分耗费资源的,而且效率低下.为了解决这个问题,WebSocket协议应运而生.它只需要进行一次握手就可以打开一条通信通道,并且不需要客户端发起查询请求,当有信息时,服务器会回调客户端.

   http与websocket区别：
   ![image](https://img-blog.csdn.net/20180510223926952?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21vc2hvd2dhbWU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

##### 1.2 业务场景 

- http协议是单工通信，只能客户端主动发送请求，服务器接收请求并返回。对于一些需要服务端主动发起的请求，http无法解决，只能通过客户端定时轮询的方式变相解决。然而轮询非常耗费网络资源，而且对于服务器端造成巨大压力。使用websocket可以很好的解决这个问题，websocket允许服务器主动发送信息给客户端。 适合的业务场景：即时通信，客服对话，在线咨询，工单系统等。

##### 1.3 websocket存在的问题      
    
- 现在，很多网站为了实现推送技术，所用的技术都是 Ajax 轮询。轮询是在特定的的时间间隔（如每1秒），
由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，
即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，
显然这样会浪费很多的带宽等资源。
HTML5 定义的 WebSocket协议，能更好的节省服务器资源和带宽，并且能够更实时地进行通讯。    
        
![image](https://www.runoob.com/wp-content/uploads/2016/03/ws.png)



#### 2代码
#####  2.1 pom.xml

<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-websocket</artifactId>
		</dependency>
		

##### 2.2  WebSocket服务端代码

    package com.kevinsun.websocketdemo;

    import java.io.IOException;  
    import java.util.concurrent.CopyOnWriteArraySet;  
    import java.util.concurrent.atomic.AtomicInteger;  
  
    import javax.websocket.OnClose;  
    import javax.websocket.OnError;  
    import javax.websocket.OnMessage;  
    import javax.websocket.OnOpen;  
    import javax.websocket.Session;  
    import javax.websocket.server.ServerEndpoint;  
  
    import org.slf4j.Logger;  
    import org.slf4j.LoggerFactory;  
    import org.springframework.stereotype.Component;  
  
    /** 
     * WebSocket服务端示例 
    * 
    */  
    @ServerEndpoint(value = "/ws/echo")  
    @Component  
    public class WebSocketServer {  
  
    private static Logger log = LoggerFactory.getLogger(WebSocketServer.class);  
    private static final AtomicInteger OnlineCount = new AtomicInteger(0);  
    // concurrent包的线程安全Set，用来存放每个客户端对应的Session对象。  
    private static CopyOnWriteArraySet<Session> SessionSet = new CopyOnWriteArraySet<Session>();  
  
  
    /** 
     * 连接建立成功调用的方法 
     */  
    @OnOpen  
    public void onOpen(Session session) {  
        SessionSet.add(session);   
        int cnt = OnlineCount.incrementAndGet(); // 在线数加1  
        log.info("有连接加入，当前连接数为：{}", cnt);  
        SendMessage(session, "连接成功");  
    }  
  
    /** 
     * 连接关闭调用的方法 
     */  
    @OnClose  
    public void onClose(Session session) {  
        SessionSet.remove(session);  
        int cnt = OnlineCount.decrementAndGet();  
        log.info("有连接关闭，当前连接数为：{}", cnt);  
    }  
  
    /** 
     * 收到客户端消息后调用的方法 
     *  
     * @param message 
     *            客户端发送过来的消息 
     */  
    @OnMessage  
    public void onMessage(String message, Session session) {  
        log.info("来自客户端的消息：{}",message);  
        SendMessage(session, "收到消息，消息内容："+message);  
  
    }  
  
    /** 
     * 出现错误 
     * @param session 
     * @param error 
     */  
    @OnError  
    public void onError(Session session, Throwable error) {  
        log.error("发生错误：{}，Session ID： {}",error.getMessage(),session.getId());  
        error.printStackTrace();  
    }  
  
    /** 
     * 发送消息，实践表明，每次浏览器刷新，session会发生变化。 
     * @param session 
     * @param message 
     */  
    public static void SendMessage(Session session, String message) {  
        try {  
            session.getBasicRemote().sendText(String.format("%s (From Server，Session ID=%s)",message,session.getId()));  
        } catch (IOException e) {  
            log.error("发送消息出错：{}", e.getMessage());  
            e.printStackTrace();  
        }  
    }  
  
    /** 
     * 群发消息 
     * @param message 
     * @throws IOException 
     */  
    public static void BroadCastInfo(String message) throws IOException {  
        for (Session session : SessionSet) {  
            if(session.isOpen()){  
                SendMessage(session, message);  
            }  
        }  
    }  
  
    /** 
     * 指定Session发送消息 
     * @param sessionId 
     * @param message 
     * @throws IOException 
     */  
    public static void SendMessage(String sessionId,String message) throws IOException {  
        Session session = null;  
        for (Session s : SessionSet) {  
            if(s.getId().equals(sessionId)){  
                session = s;  
                break;  
            }  
        }  
        if(session!=null){  
            SendMessage(session, message);  
        }  
        else{  
            log.warn("没有找到你指定ID的会话：{}",sessionId);  
        }  
    }  
      
}  
		

##### 2.3 服务端推送消息测试Controller 
    
    package com.kevinsun.websocketdemo;

    import java.io.IOException;  

    import org.springframework.web.bind.annotation.RequestMapping;  
    import org.springframework.web.bind.annotation.RequestMethod;  
    import org.springframework.web.bind.annotation.RequestParam;  
    import org.springframework.web.bind.annotation.RestController;  
  
    /** 
     * WebSocket服务器端推送消息示例Controller 
     *  
     * 
     */  
    @RestController  
    @RequestMapping("/api/ws")  
    public class WebSocketController {  
  
    @RequestMapping(value="/sendAll", method=RequestMethod.GET)  
    /** 
     * 群发消息内容 
     * @param message 
     * @return 
     */  
    String sendAllMessage(@RequestParam(required=true) String message){  
        try {  
            WebSocketServer.BroadCastInfo(message);  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        return "ok";  
    }  
    @RequestMapping(value="/sendOne", method=RequestMethod.GET)  
    
    /** 
     * 指定会话ID发消息 
     * @param message 消息内容 
     * @param id 连接会话ID 
     * @return 
     */  
    String sendOneMessage(@RequestParam(required=true) String message,@RequestParam(required=true) String id){  
        try {  
            WebSocketServer.SendMessage(id,message);  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        return "ok";  
    }  
}  


##### 2.4 开启websocket支持
    
    package com.kevinsun.websocketdemo;

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

##### 2.5 页面html代码

    <!DOCTYPE html>
    <!--   
    功能：WebSocket使用示例  
    -->
    <html>
    <head>
    <meta charset="UTF-8">
    <title>websocketdemo测试</title>
    <style type="text/css">
    h3, h4 {
	text-align: center;
    }
    </style>
    </head>
    <body>

	<h3>WebSocketDemo测试</h3>
	<h4>单发消息触发链接:/api/ws/sendOne?message=单发消息内容&id=none</h4>
	<h4>群发消息触发链接:/api/ws/sendAll?message=群发消息内容</h4>


	<script type="text/javascript">
		var socket;
		if (typeof (WebSocket) == "undefined") {
			console.log("遗憾：您的浏览器不支持WebSocket");
		} else {
			console.log("恭喜：您的浏览器支持WebSocket");

			//实现化WebSocket对象  
			//指定要连接的服务器地址与端口建立连接   
			//注意ws、wss使用不同的端口。我使用自签名的证书测试，  
			//无法使用wss，浏览器打开WebSocket时报错  
			//ws对应http、wss对应https。  
			socket = new WebSocket("ws://localhost:8080/ws/echo");
			//连接打开事件    
			socket.onopen = function() {
				console.log("Socket 已打开");
				socket.send("消息发送测试(From Client)");
			};
			//收到消息事件    
			socket.onmessage = function(msg) {
				console.log(msg.data);
				alert("websocket服务端推送消息:"+msg.data);
			};
			//连接关闭事件    
			socket.onclose = function() {
				console.log("Socket已关闭");
			};
			//发生了错误事件    
			socket.onerror = function() {
				alert("Socket发生了错误");
			}

			//窗口关闭时，关闭连接  
			window.unload = function() {
				socket.close();
			};
		}
	</script>
    </body>
    </html>


#### 3 demo

##### 3.1 启动项目后，打开页面

http://localhost:8080/hello.html

##### 3.2 发送消息

单发消息触发链接:/api/ws/sendOne?message=单发消息内容&id=none

群发消息触发链接:/api/ws/sendAll?message=群发消息内容


#### 4 Github

github：[https://github.com/kevinsun2010/websocket-kevinsun-leaning](https://note.youdao.com/)
