

参考链接：
[万字详解，带你彻底掌握 WebSocket 用法（至尊典藏版） (qq.com)](https://mp.weixin.qq.com/s/0SI9C3tBPqyIeMtkfTZk8g)



# 正文：


### 与http的区别


***不能使用浏览器地址栏直接访问，因为他本质上时对http的封装

#### 1，消息格式
HTTP报文是面向文本的，报文中的每一个字段都是一些ASCII码串，各个字段的长度是不确定的。
WebSocket 的消息格式可以是文本或二进制数据，并且 WebSocket 消息的传输是在一个已经建立的连接上进行的，因此不需要再进行 HTTP 请求和响应的握手操作。


WebSocket 消息格式由两个部分组成：消息头和消息体。

消息头包含以下信息：

- **FIN：** 表示这是一条完整的消息，一般情况下都是1。
    
- **RSV1、RSV2、RSV3：** 暂时没有使用，一般都是0。
    
- **Opcode：** 表示消息的类型，包括文本消息、二进制消息等。
    
- **Mask：** 表示消息是否加密。
    
- **Payload length：** 表示消息体的长度。
    
- **Masking key：** 仅在消息需要加密时出现，用于对消息进行解密。
    

消息体就是实际传输的数据，可以是文本或二进制数据。
#### 2，发送、接收
![image.png](http://obimage.wenzhuo4657.cn/20240408205021.png)


#### 3，通信流程

握手：
在计算机网络和通信协议的上下文中，握手（Handshake）是一种约定机制，它是两个终端之间建立可靠连接或交换必要信息的过程。对于WebSocket协议来说，握手是指在建立WebSocket连接前，客户端与服务器之间进行的一种特殊交互过程，目的是协商并确认双方都能正确地按照WebSocket协议进行通信。

下图表示了一次握手

![image.png](http://obimage.wenzhuo4657.cn/20240408210512.png)


握手操作

![image.png](http://obimage.wenzhuo4657.cn/20240408205747.png)

![image.png](http://obimage.wenzhuo4657.cn/20240408205952.png)

### 使用


#### js实现客户端
```
<!DOCTYPE html>  
<html>  
<head>  
<meta charset="UTF-8">  
<title>WebSocket</title>  
</head>  
<body>  
<script type="text/javascript">  
let websocket = new WebSocket("ws://localhost:8080/rs");  
  
// 连接断开  
websocket.onclose = e => {  
console.log(`连接关闭: code=${e.code}, reason=${e.reason}`)  
}  
// 收到消息  
websocket.onmessage = e => {  
console.log(`收到消息：${e.data}`);  
}  
// 异常  
websocket.onerror = e => {  
console.log("连接异常")  
console.error(e)  
}  
// 连接打开  
websocket.onopen = e => {  
console.log("连接打开");  
websocket.send("创建成功");  
}  
</script>  
</body>  
</html>

```


#### 基于java注解实现服务器端


服务端代码
```
package cn.wenzhuo4657.echo.server;  
  
import cn.wenzhuo4657.Service.useService;  
import jakarta.websocket.*;  
import jakarta.websocket.server.ServerEndpoint;  
import org.slf4j.Logger;  
import org.slf4j.LoggerFactory;  
import org.springframework.beans.BeansException;  
import org.springframework.context.ApplicationContext;  
import org.springframework.context.ApplicationContextAware;  
import org.springframework.scheduling.annotation.Scheduled;  
import org.springframework.stereotype.Component;  
  
import java.io.IOException;  
import java.util.concurrent.ConcurrentHashMap;  
  
/**  
* @className: echoServer  
* @author: wenzhuo4657  
* @date: 2024/4/2 14:07  
* @Version: 1.0  
* @description: 监听客服端连接  
*/  
  
@ServerEndpoint(value = "/rs")  
@Component  
public class echoServer implements ApplicationContextAware {  
private Logger logger= LoggerFactory.getLogger(echoServer.class);  
;  
private Session session;  
  
private static ApplicationContext applicationContext;//可以这个属性连接spring容器，操作bean的注入等，  
private String userId;  
// 客户端连接存储：可通过session向客户端操作  
private static ConcurrentHashMap<String,Session> sessionPool = new ConcurrentHashMap<String,Session>();  
  
@Override  
public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {  
echoServer.applicationContext = applicationContext;  
}  
  
  
private useService useService;  
  
  
//执行一些初始化工作，每个实例仅执行一次  
@OnOpen  
public void open(Session session){  
this.session=session;  
this.userId=session.getId();  
sessionPool.put(userId,session);  
  
  
// useService=echoServer.applicationContext.getBean(useService.class);  
// useService.start();  
logger.info("[websocket] 新的连接：id={}", this.session.getId());  
// logger.info("[websocket] 用户id={}",userId);  
}  
  
@OnClose  
public void onclose(){  
sessionPool.remove(userId);  
logger.info("[websocket] 连接关闭 用户id={}",userId);  
}  
  
  
@OnError  
public void onError(Throwable error) {  
  
logger.error("用户错误,原因:"+error.getMessage());  
error.printStackTrace();  
}  
  
@OnMessage  
public String onMessage(String text){  
logger.info("已收到：{}",text);  
return "收到";  
}  
  
@Scheduled(fixedRate = 2000)  
public void sendMsg() throws IOException {  
for (Session session:sessionPool.values()){  
session.getBasicRemote().sendText("心跳");  
}  
  
}  
}
```

配置类
```
@Configuration  
public class WebSocketConfiguration {  
  
@Bean  
public ServerEndpointExporter serverEndpointExporter(){  
return new ServerEndpointExporter();//该对象的注入会使spring容器开启对服务端实例的扫描，  
}  
}
```


#### 基于spring内置类实现服务器端


![sdfdsf](http://obimage.wenzhuo4657.cn/20240409203619.png)


### 常见应用场景


![image.png](http://obimage.wenzhuo4657.cn/20240410134908.png)
