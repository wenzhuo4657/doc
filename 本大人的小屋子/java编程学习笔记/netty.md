而netty是一个Java网络编程框架，支持多种协议的开发，比如TCP、UDP、HTTP、WebSocket等。我们可以利用netty实现websocket通信，也可以用其他的。



应用场景
- 人工智能物联网（AIOT）：设备通信吞吐量、并发量巨大，基本日均TB级别，对于频繁的通信，例如心跳机制，netty可以解决此类频繁的通信IO操作。
- 互联网行业：RocketMQ、Dubbo底层的基本通信组件就是netty，



demo:https://github.com/wenzhuo4657/demo/tree/main/Netty


# 基本概念
## epoll

实际上就是不用去遍历channel得到发生事件的列表，而是提出了就绪列表的概念去处理事件，将channel（表示通道）和就绪操作（表示某通道有事件发生）解耦。
![image.png](http://obimage.wenzhuo4657.cn/20240422185839.png)


## Reactor模型
注意：连接操作简单，交由主线程处理即可。

单线程：一种处理大量并发I/O请求的高效方法，不同于nio线程模型，nio是一种允许程序在没有数据可处理的情况下不被阻塞
![image.png](http://obimage.wenzhuo4657.cn/20240422194947.png)

多线程：交由线程池处理

![image.png](http://obimage.wenzhuo4657.cn/20240422195530.png)


多主多从：多个Reactor，每个Reactor管理一个线程池
![image.png](http://obimage.wenzhuo4657.cn/20240422200246.png)

netty 线程模型
![image.png](http://obimage.wenzhuo4657.cn/20240423124737.png)
![image.png](http://obimage.wenzhuo4657.cn/20240423124707.png)



# 核心组件

- Bootstrap：配置Netty程序，串联Netty的各个组件。
    Bootstrap: 客户端的启动程序
    ServerBootstrap:服务端启动程序
-  ChannelFuture 
    Netty 中的所有 I/O 操作都是异步的。这意味着任何 I/O 呼叫都将立即返回，但不能保证请求的 I/O 操作在呼叫结束时已完成。相反，您将返回一个 ChannelFuture 实例，该实例为您提供有关 I/O 操作的结果或状态的信息。
- channel
  主要实现类:
  NioSocketChannel:异步的客户端TCPSocket连接通道
  NioServerSocketChannel:异步的服务端TCPSocket连接通道
  NioDatagramChannel:异步的UDP连接通道
  NioSctpChannel:异步的客户端Sctp连接通道
  NioSctpServerChannel:异步的服务端Sctp连接通道
- NioEventLoop
  内部维护了一个线程和任务队列，支持异步提交执行任务。当线程启动时会调用NioEventLoop的
  run方法来执行io任务或非io任务。
  io任务:如accept、connect、read、write事件等，由processSelectedKeys方法触发。
  非io任务:如register0、bind0等任务将会被添加到taskQueue任务队列中，由runAllTasks方法触发
- NioEventLoopGroup :管理NioEventLoop的生命周期
- ByteBuf: 
- channelHandler
- channelHandlerContenxt
- channelPipline：双向链表，表征channelHandler依次调用


# Netty编解码


![image.png](http://obimage.wenzhuo4657.cn/20240423133634.png)

![image.png](http://obimage.wenzhuo4657.cn/20240423135701.png)
pipeline会根据其handler类型是出站还是入站决定是否执行

![image.png](http://obimage.wenzhuo4657.cn/20240423140126.png)


*其他编解码器

| 编解码器          |              |
| ------------- | ------------ |
| ObjectEncoder |              |
| ObjectDecoder |              |
| Protobuf      |              |
| Protostuff    | 基于Protobuf实现 |

  
```
<!-- https://mvnrepository.com/artifact/com.dyuproject.protostuff/protostuff-core -->
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.0.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.dyuproject.protostuff/protostuff-api -->
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-api</artifactId>
    <version>1.0.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.dyuproject.protostuff/protostuff-runtime -->
<dependency>
    <groupId>com.dyuproject.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.0.1</version>
</dependency>
```


# 粘包和拆包
tcp等流式传输没有定义数据的边界，也不能理解业务数据的具体含义，有可能会将一条完整数据拆包成多个数据包，或者将多条完整数据粘包成一个数据包。
tcp协议在在传输数据时，是根据tcp缓冲区的使用情况进行数据包的划分的。

解决方案：为业务数据设置边界


Netty为粘包和拆包提供了多个解码器，每个解码器配有相应的分包解决方案。
- LineBasedFrameDecoder:回车换行分包，以回车换行为分包的依据
    pipeline.addLast(new LineBasedFrameDecoder(1024));
- DelimiterBasedFrameDecoder:特殊分隔符分包，以指定的特殊分隔符为分包依据，局限是消息内容中不能出现特殊符。
  pipeline.addLast(new DelimiterBasedFrameDecoder(1024，Unpooled.copiedBuffer("".getBytes())));
- FixedLengthFrameDecode:固定长度报文分包，消息长度被指定，不足的以空格补足，
  pipeline.addLast(new FixedLengthFrameDecoder(1024));

也可以使用自定义分包解码器，这也是较为常用的解决方案。
自定义分包往往在发送每条数据的时候，将数据的长度一并发送。比如发送的数据的前4个字节用来表示消息的实
际长度，之后根据消息的实际长度来获得具体的数据


![image.png](http://obimage.wenzhuo4657.cn/20240425112936.png)




# 心跳机制

# 断线重连
1，连接失败后重连
2，连接成功后，channel意外关闭重连


![image.png](http://obimage.wenzhuo4657.cn/20240425184905.png)
