### 概念
WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。

### 基础环境
快速搭建Spring框架，我们使用Spring boot，这里先不讨论SpringBoot，只知道它是一个“快速搭建Spring项目的一站式解决方案”就OK了。
要使用Spring的WebSocket功能，我们需要添加依赖：
```
<dependency>    
        <groupId>org.springframework.boot</groupId>    
        <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>

```

### 相关配置

```
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {    
    @Override    
    public void registerStompEndpoints(StompEndpointRegistry registry) {        
        registry.addEndpoint("/socket").withSockJS();       
    }    
    @Override    
    public void configureMessageBroker(MessageBrokerRegistry registry) {  
        registry.enableSimpleBroker("/topic");      
        registry.setApplicationDestinationPrefixes("/app"); 
    }
}


```

> 相关说明：
- `registerStompEndpoints(StompEndpointRegistry registry)`  
这个方法的作用是添加一个服务端点，来接收客户端的连接。
- `registry.addEndpoint("/socket")`  
表示添加了一个/socket端点，客户端就可以通过这个端点来进行连接。
- `registry.enableSimpleBroker("/topic")`  
表示客户端订阅地址的前缀信息，也就是客户端接收服务端消息的地址的前缀信息（比较绕，看完整个例子，大概就能明白了）
- registry.setApplicationDestinationPrefixes("/app")  
指服务端接收地址的前缀，意思就是说客户端给服务端发消息的地址的前缀

### 编写后台业务  

- MessageMapping  
Spring对于WebSocket封装的特别简单，提供了一个@MessageMapping注解，功能类似@RequestMapping，它是存在于Controller中的，定义一个消息的基本请求，功能也跟@RequestMapping类似，包括支持通配符``的url定义等等，详细用法参见Annotation Message Handling

- SimpMessagingTemplate  
SimpMessagingTemplate是Spring-WebSocket内置的一个消息发送工具，可以将消息发送到指定的客户端。
```
@Controller
public class GreetingController {    
    @Resource
    private SimpMessagingTemplate simpMessagingTemplate;    
    @RequestMapping("/helloSocket")    
    public String index(){        
        return "/hello/index";    
    }    
    @MessageMapping("/change-notice")    
    public void greeting(String value){
        this.simpMessagingTemplate.convertAndSend("/topic/notice", value);    
    }
}

```

1. index()
指定了一个页面，用来实现WebSocket客户端发送公告功能，使用的是@RequestMapping，所以接收的是http请求，进行页面跳转。
2. greeting(String value)
这个方法是接收客户端发送功公告的WebSocket请求，使用的是@MessageMapping。

3. this.simpMessagingTemplate.convertAndSend("/topic/notice", value)
这个方法官方给出的解释是Convert the given Object to serialized form, possibly using a MessageConverter, wrap it as a message and send it to the given destination. 意思就是“将给定的对象进行序列化，使用‘MessageConverter’进行包装转化成一条消息，发送到指定的目标”，通俗点讲就是我们使用这个方法进行消息的转发发送！


前面我们全局配置中指定了服务端接收的连接以 app大头，所以客户端发送公告的请求连接应该是/app/change-notice。
服务端代码就这么简单，跟写SpringMVC类似，同样上面的geeting(String value)方法我们还可以使用另一个注解@SendTo换成另一种写法。

```
@MessageMapping("/change-notice")
@SendTo("/topic/notice")
public String greeting(String value) {    
    return value;
}
```

- @SendTo定义了消息的目的地。结合例子解释就是“接收/app/change-notice发来的value，然后将value转发到/topic/notice客户端。

- /topic/notice是客户端发起连接后，订阅服务端消息时指定的一个地址，用于接收服务端的返回，后面我们在写客户端代码的时候会看见。


### 前端

- SockJS：  
SockJS 是一个浏览器上运行的 JavaScript 库，如果浏览器不支持 WebSocket，该库可以模拟对 WebSocket 的支持，实现浏览器和 Web 服务器之间低延迟、全双工、跨域的通讯通道。

- Stomp  
Stomp 提供了客户端和代理之间进行广泛消息传输的框架。Stomp 是一个非常简单而且易用的通讯协议实现，尽管代理端的编写可能非常复杂，但是编写一个 Stomp 客户端却是很简单的事情，另外你可以使用 Telnet 来与你的 Stomp 代理进行交互。


```
<div>    
    <div>        
        <button id="connect" onclick="connect();">Connect</button> 
       <button id="disconnect" disabled="disabled" onclick="disconnect();">Disconnect</button>    
    </div>    
    <div id="conversationDiv">        
        <p>            
            <label>notice content?</label>        
        </p>        
        <p>            
              <textarea id="name" rows="5"></textarea>        
        </p>        
        <button id="sendName" onclick="sendName();">Send</button>        
        <p id="response"></p>    
    </div>
</div>


<script src="/js/sockjs-0.3.4.min.js"></script>
<script src="/js/stomp.min.js"></script>
<script>    
    var stompClient = null;    
    function setConnected(connected) {        
        document.getElementById('connect').disabled = connected;        
        document.getElementById('disconnect').disabled = !connected;        
        document.getElementById('conversationDiv').style.visibility = connected ? 'visible' : 'hidden';        
        document.getElementById('response').innerHTML = '';    
    }    
    // 开启socket连接
    function connect() {        
        var socket = new SockJS('/socket');        
        stompClient = Stomp.over(socket);        
        stompClient.connect({}, function (frame) {            
             setConnected(true);            
        });    
    }    
    // 断开socket连接
    function disconnect() {        
        if (stompClient != null) {            
            stompClient.disconnect();        
        }        
        setConnected(false);        
        console.log("Disconnected");    
    }    
    // 向‘/app/change-notice’服务端发送消息
    function sendName() {        
        var value = document.getElementById('name').value;            
        stompClient.send("/app/change-notice", {}, value);    
    }    
    connect();
</script>

```

### 开启Socket

```
var socket = new SockJS('/socket'); 先构建一个SockJS对象

stompClient = Stomp.over(socket); 用Stomp将SockJS进行协议封装

stompClient.connect() 与服务端进行连接，同时有一个回调函数，处理连接成功后的操作信息。

stompClient.send("/app/change-notice", {}, value);
发送消息
```
```
订阅（监听）

var noticeSocket = function () {    
  var s = new SockJS('/socket');    
  var stompClient = Stomp.over(s);    
  stompClient.connect({}, function () {         
    console.log('notice socket connected!');
    stompClient.subscribe('/topic/notice', function (data) {            
      $('.message span.content').html(data.body);        
    });    
 });
};

```
> 相关说明：
这里与发送公告代码不同的是，在stompClient建立连接成功之后，我们要监听服务端发送过来的信息，接收到之后，改变页面上公告的内容，所以用到了stompClient.subscribe()
这个subscribe()方法功能就是定义一个订阅地址，用来接收服务端的信息（在服务端代码中，我们在@SendTo中指定了这个订阅地址“/topic/notice”），当收到消息后，在回调函数中处理业务逻辑。


### nginx

```
location ^~ /socket {
        # proxy_pass http://dockerhost:8089;
        proxy_pass http://websocket;
        # proxy_pass http://145.170.23.128:8089;
        proxy_http_version 1.1; 
        proxy_set_header Upgrade $http_upgrade; 
        proxy_set_header Connection "upgrade";
        proxy_set_header Origin "";
        # proxy_set_header Connection $connection_upgrade;
      }



```

### 参考资料
[spring 官网](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#websocket)

[简书教程-后端](https://www.jianshu.com/p/60799f1356c5)

[简书教程-前端](https://www.jianshu.com/p/8500ad65eb50)

[GitHub](https://github.com/myitroad/spring-in-action-4/tree/master/Chapter_18)