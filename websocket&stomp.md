# Websocket&Stomp

## WebSocket&SockJs简介

1. WebSocket是两个应用之间全双工的通信通道，
2. WebSocket 是底层协议，SockJS 是WebSocket 的模拟兼容方案，也是底层协议；而 STOMP 是基于 WebSocket（SockJS） 的上层通讯协议
3. SockJS 会优先选择 WebSocket 协议，通常引入sockJs包。

##Stomp

1. STOMP在WebSocket之上提供了一个基于帧的线路格式（frame-based wire format）层，用来定义消息的语义。可以能路由信息到指定消息地点、直接使用成熟的STOMP代理进行广播 如:RabbitMQ, ActiveMQ。
2. STOMP帧由命令、一个或多个头信息以及负载所组成：![](E:\学习总结\spring系列\stomp帧.jpg)

# springboot-WebSocket

##Spring抽象

1. WebSocketHandler接口是消息处理器（单例Singleton），WebSocketSession是底层Session的高层抽象![](E:\学习总结\spring系列\WebSocketHandler体系.jpg)

2. 握手处理HandshakeHandler.doHandshake(request,response,wshandler,attributes) --->DefaultHandshakeHandler，一般它就足够无需复写。

   - 不使用secutity需复写determineUser方法获取Principal

3. 握手拦截器  HandshakeInterceptor.beforeHandshake握手前&afterHandshake握手后-->HttpSessionHandshakeInterceptor：

   - 校验是否连接（权限等）
   - request.getPrincipal()获取当前用户  **security可自动设Principal，否则**DefaultHandshakeHandler重写determineUser方法获取当前登录用户

4. @Configuration实现WebSocketConfigurer接口registerWebSocketHandlers方法注册WebSocketHandler处理器，一般是TextWebSocketHandler。激活@EnableWebSocket

5. ```java
   public class MySocketHandler extends TextWebSocketHandler {
       private String x= UUID.randomUUID().toString();	//单例，每个此x都一样
       @Override  //连接建立后
       public void afterConnectionEstablished(WebSocketSession session) throws Exception {
           System.out.println(session.getId());
       }
       @Override	//收到消息处理
       protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
           session.sendMessage(new TextMessage("kkkkkk"+UUID.randomUUID().toString()));
       }
   
       @Override   	//连接错误
       public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
           System.out.println("onerror");
       }
   
       @Override
       public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
           System.out.println("onclose");
       }
       @Override	//是否支持消息分隔
       public boolean supportsPartialMessages() {
           return false;
       }
   }
   ```

6. ```java
   @Configuration
   @EnableWebSocket
   public class WebSocketConfig implements WebSocketConfigurer {
       @Override
       public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
           registry.addHandler(new MySocketHandler(), "/c/{id}").setAllowedOrigins("*").addInterceptors(new HttpSessionHandshakeInterceptor(){
               @Override
               public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
                   return super.beforeHandshake(request, response, wsHandler, attributes);
               }
   
               @Override
               public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception ex) {
                   super.afterHandshake(request, response, wsHandler, ex);
               }
           }).setHandshakeHandler(new DefaultHandshakeHandler(){
               @Override
               protected Principal determineUser(ServerHttpRequest request, WebSocketHandler wsHandler, Map<String, Object> attributes) {
                   //无Security需重写它获取Principal
                   return super.determineUser(request, wsHandler, attributes);
               }
           });
       }
   ```

## 兼容tomcat方案

1. 引入ServerEndpoint扫描，激活@EnableWebSocket，将@ServerEndpoint作为@Component注册

   ```java
   @Bean
   public ServerEndpointExporter serverEndpointExporter() {
       return new ServerEndpointExporter();  //扫描@ServerEndpoint注解的组件
   }
   ```

2. @ServerEndpoint设路径,@OnOpen连接建立,@OnClose连接关闭,@OnError连接错误,@OnMessage收到信息

3. @ServerEndpoint是Prototype，其成员变量分别属于对应session。**每建立新连接自动创建对应的新对象**

4. spring代码

   ```java
   @Component
   @ServerEndpoint("/w/{id}")
   public class WEndPoint {
    private static ConcurrentHashMap<String,Session> webSocketMap = new ConcurrentHashMap<>();//存储所有session   
       @OnOpen
       public void onOpen(Session session,EndpointConfig endpointConfig) {
       }
   
       /**
        * 连接关闭调用的方法
        */
       @OnClose
       public void onClose(Session session,CloseReason closeReason) {;
       }
   
       /**
        * 收到客户端消息后调用的方法
        * @param message 客户端发送过来的消息
        */
       @OnMessage
       public void onMessage(String message, Session session, @PathParam("id") String id) {
               session.getBasicRemote().sendText(message);
       }
   
       @OnError
       public void onError(Session session, Throwable error) {
       }
   
   }
   ```

## JS代码

<script>
    //ws开头
    socket = new WebSocket("ws://localhost:8080/c/ljh");
    socket.onopen = function (event) {
        console.log("websocket已打开");
    };
    //获得消息事件
    socket.onmessage = function (msg) {
        s.textContent = msg.data
    };
    //关闭事件
    socket.onclose = function () {
        console.log("websocket已关闭");
    };
    //发生了错误事件
    socket.onerror = function () {
        console.log("websocket发生了错误");
    }
//发消息
    function f() {
        socket.send(Math.random().toString())
    }
</script>

# springboot-stomp

## 结构体系

![stomp结构图](E:\学习总结\spring系列\stomp结构图.png)

1. ApplicationDestinationPrefixes：发往@MessageMapping的前缀，一般/app；UserDestinationPrefix：一对一发送前缀，默认/user；Broker：消息代理队，SimpleBroker基于内存的简单代理，可利用rabbilmq

2. 广播：stompClient.send("/app/pe", {}, body) ->@MessageMapping("/pe")->@SendTo("/topic/xxx")->stompClient.subscribe('/topic/xxx',callback)   所有订阅/topic/xxx的浏览器全都收到

3. 单播：stompClient.send("/app/pe", {}, body) ->@MessageMapping("/pe")->@SendToUser("/topic/xxx")->stompClient.subscribe(/user'/topic/xxx',callback)  或"/user/用户名/topic/xxx"      哪个订阅者发送返回给谁

4. 消息目的地解析：spring将根据前缀自动解析  

   - ①/app取出找到对应@MessageMapping   

   - ②/user/topic/xxx和/user/当前Principal用户名/topic/xxx--- ->/user/topic/xxx-user后跟sessionId

     

5. **SimpMessagingTemplate**在程序任何位置发送任意类型信息

   1. 广播：SimpMessagingTemplate.convertAndSen("目的地",payload)
   2. 单播：SimpMessagingTemplate.convertAndSendToUser("发哪个用户名","目的地无/user前缀",payload)

6. 相关注解

   - @MessageMapping+@SendTo+stompClient.subscribe('广播',callback)：广播
   - @MessageMapping+@SendToUser+stompClient.subscribe('/user/**',callback)：单播    
   - @SendToUser和/user/topic/xxx配套使用
   - @SubscribeMapping：一次性异步订阅，每次连接只触发一次类似http，用处不大
   - **@MessageMapping返回值会被发送到消息代理，默认添加/topic前缀。@SendTo 重写目的地；    **
   - **@SubscribeMapping返回值会直接发送到客户端，不经过代理。如果加上@SendTo 注解的话，则要经过消息代理。**

7. 常用@Controller的Mapping参数

   - Principal参数自动填充为本用户  Message完整消息    消息标头   StompHeaderAccessor消息头处理包装
   - @MessageExceptionHandler 异常处理方法    @Payload消息负载，默认匹配它，一般无需设置
   - @Header某个消息头     @Headers整体消息头  Map类型     @DestinationVariable路径变量 

   

8. 

## 代理介绍

1. @Configuration实现WebSocketMessageBrokerConfigurer接口registerStompEndpoints和configureMessageBroker，激活@EnableWebSocketMessageBroker

2. ```java
   @Configuration
   @EnableWebSocketMessageBroker
   public class WebSocketBrokerConfig implements WebSocketMessageBrokerConfigurer {
       @Override
       public void registerStompEndpoints(StompEndpointRegistry registry) {
     registry.addEndpoint("/portfolio").setAllowedOrigins("*").setHandshakeHandler(new DefaultHandshakeHandler() {
               @Override   //无security则重写determineUser方法
               protected Principal determineUser(ServerHttpRequest request, WebSocketHandler wsHandler, Map<String, Object> attributes) {
                   return new
               }
           });
       }
       //校验JS的connect的header携带token
       @Override
       public void configureClientInboundChannel(ChannelRegistration registration) {
           registration.interceptors(new ChannelInterceptor() {
               @Override
               public Message<?> preSend(Message<?> message, MessageChannel channel) {
                   StompHeaderAccessor accessor =
                           MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                   if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                       Authentication user = ... ; // access authentication header(s)
                       accessor.setUser(user);
                   }
                   return message;
               }
           });
           
       @Override
       public void configureMessageBroker(MessageBrokerRegistry config) {
           config.enableSimpleBroker("/topic");
           config.setUserDestinationPrefix("/user");
           config.setApplicationDestinationPrefixes("/app");
       }
   }
   ```

## JS代码

```javascript
socket = new WebSocket("ws://localhost:8080/portfolio");
var stompClient = Stomp.over(socket);
//client.connect(headers对象, connectCallback, errorCallback);
stompClient.connect({a:x}, function (frame) {
    //单播            //header可携带token
    stompClient.subscribe('/user/topic/greetings', function (greeting) {
        s.innerHTML = JSON.parse(greeting.body).content
    });
//广播
    stompClient.subscribe('/topic/gr', function (greeting) {
        s.innerHTML = JSON.parse(greeting.body).content
    });
});

function f() {
    //发送消息
    stompClient.send("/app/pe", {}, JSON.stringify({'name': Math.random().toString()}));
}

```

## 3

