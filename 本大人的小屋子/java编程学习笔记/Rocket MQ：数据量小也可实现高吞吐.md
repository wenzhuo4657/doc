参考：[MQ系列3：RocketMQ 架构分析 - Hello-Brand - 博客园](https://www.cnblogs.com/wzh2010/p/16556570.html)



MQ(消息队列)，解决并不是消息的先后问题，而是由于网络导致的通信问题，


nameserver: rokcetmq的服务注册中心，管理broker节点并向生产者和消费者提供路由等功能。
broker:真正存储消息的服务单元，部署时有只读、读写、两主两从等架构。


![image.png](http://obimage.wenzhuo4657.cn/20240502103011.png)


![image.png](http://obimage.wenzhuo4657.cn/20240504175957.png)

# 命令学习



- `nohup`：命令用于忽略所有挂起（SIGHUP）信号，通常当用户退出终端时，所有由该终端启动的进程都会收到SIGHUP信号从而导致进程结束。使用`nohup`可以避免这种默认行为，使得进程在终端关闭后仍能继续运行。
- `&`：最后的`&`字符将命令放到后台执行，使得命令执行后终端可以立刻返回到命令提示符，而不是等待该进程结束。


检验成功启动命令
```
ps aux | grep mqbroker
ps aux | grep mqname
```


启动nameserver ,并指定ip和端口
[root@localhost bin]# nohup ./mqnamesrv -n 192.168.213.130:9876 &



启动broker注册到指定nameserver
nohup ./mqbroker -n 192.168.213.130:9876  &



执行tools.sh

![image.png](http://obimage.wenzhuo4657.cn/20240502174458.png)

启动生产者
 ./tools.sh  org.apache.rocketmq.example.quickstart.Producer

启动消费者
./tools.sh org.apache.rocketmq.example.quickstart.Consumer



关闭服务器
./mqshutdown broker

./mqshutdown namesrv


查看有哪些集群以及相关信息
./mqadmin clusterList

创建topic
./mqadmin updateTopic -n 192.168.213.132:9876 -c DefaultCluster -t myTopic1 








指定配置文件启动

![image.png](http://obimage.wenzhuo4657.cn/20240503125700.png)

# 基本概念

[rocketMQ (apache.org）官方文档](https://rocketmq.apache.org/docs/4.x/consumer/01concept2#load-balancing)  

## broker集群

- 两主两从*异步*
 优先：吞吐量大，缺点：由于是异步，所以可能丢失消息
 配置文件： conf/2m-2s-async
- 两主两从*同步*
 优先：消息安全投递，不会丢失消息，缺点：吞吐量没有异步大，会阻塞等待同步数据
 配置文件： conf/2m-2s-sync
 - 两主无从
 容易单点故障 
 配置文件： conf/2m-noslave
- Dleger高可用集群
上述三种官方提供的集群没办法实现高可用，即在master节点挂掉后，slave节点没办法自动被选举为新的master，而需要人工实现。

RocketMQ在4.5版本之后引入了第三方的Dleger高可用集群。


## topic
Topic代表了消息的主题或者分类，订阅者可以根据这些主题来接收他们感兴趣的消息。
如果尝试重新定义相同名称的Topic对象，会导致"Object already exists"的错误。


## message
### 消息发送流程：
- 生产者每 30 秒向 NameServer 拉取路由信息，Broker 每 30 秒向 NameServer 发送路由信息。
- 生产者发送消息时，会先在本地查询 Topic 路由信息。
- 如果查询不到，会请求 NameServer 查询。
- 随后将消息发送给 Broker。
- Broker 也会在本地查询 Topic 路由信息来检查消息的 Topic 是否存在。
- 随后保存消息，如果是异步发送则直接返回，如果同步发送则等待消息保存成功后返回。



### 消息分类

#### 简单消息

| MessageCate | des                                    | 是否接收响应 |
| ----------- | -------------------------------------- | ------ |
| 同步消息        | 每次发送消息必须等待响应结果后才能继续发送，换言之，等待过程中生产者是阻塞的 | yes    |
| 异步消息        | 无需等待响应结果，不阻塞                           | yes    |
| 单向消息        | 只管发送，不需要接收响应，类似于UDP                    | no     |





#### 顺序消息
该顺序指的是**消费者消费消息的顺序**按照**生产者发送消息的顺序**，分为*局部顺序和全局顺序*

**局部顺序**
实质上时保证了生产者将消息推送至队列内部的有序，并没有解决消费者消费何种队列，消费队列的顺序问题

![image.png](http://obimage.wenzhuo4657.cn/20240504145629.png)




**全局顺序**
生产者只会将消息推送至一个消息队列
缺点：会损失集群，失去高性能高可用等优势
![image.png](http://obimage.wenzhuo4657.cn/20240504145515.png)




#### 延迟消息

#### 批量消息

#### 事务消息



### 邮件过滤
在broker处实行的过滤，broker只会将consumer需要的消息类型发送，、
1,设置条件表达式过滤
![image.png](http://obimage.wenzhuo4657.cn/20240504160316.png)

2，设置sql过滤
![image.png](http://obimage.wenzhuo4657.cn/20240504160303.png)





# 消息存储

![image.png](http://obimage.wenzhuo4657.cn/20240504182412.png)



CommitLog：
消息主体以及元数据的存储主体，存储Producer端写⼊的消息主体内容,消息内容不
是定⻓的。单个⽂件⼤⼩默认1G ，⽂件名⻓度为20位，左边补零，剩余为起始偏
移量，⽐如00000000000000000000代表了第⼀个⽂件，起始偏移量为0，⽂件⼤⼩
为1G=1073741824；当第⼀个⽂件写满了，第⼆个⽂件为00000000001073741824，
起始偏移量为1073741824，以此类推。消息主要是顺序写⼊⽇志⽂件，当⽂件满
了，写⼊下⼀个⽂件；
ConsumeQueue：
消息消费队列，引⼊的⽬的主要是提⾼消息消费的性能，
IndexFile：
IndexFile（索引⽂件）提供了⼀种可以通过key或时间区间来查询消息的⽅法。



*⻚缓存与内存映射*
**⻚缓存（PageCache)是OS对⽂件的缓存，速度几乎接近于内存读写的速度。**


写⼊：OS会先写⼊⾄Cache内，异步将Cache内的数据刷盘⾄物理磁盘上。
读取：如果⼀次读取⽂件时出现未命中PageCache的情况，OS从物理磁盘上访问读取⽂件的同时，会顺序**对其他相**
**邻块的数据⽂件进⾏预读取。**
![image.png](http://obimage.wenzhuo4657.cn/20240504182945.png)


内存映射

使用MappedByteBuffer:将磁盘文件内容映射到虚拟内存中，无需通过io流读写，效率极高

消息刷盘

![image.png](http://obimage.wenzhuo4657.cn/20240504184212.png)

![image.png](http://obimage.wenzhuo4657.cn/20240504184140.png)


# 幂等消息

幂等性：多次操作结果一致

在消息队列中的幂等性体现：

- ⽣产者重复发送：
由于⽹络抖动，导致⽣产者没有收到broker的ack⽽再次重发
消息。
- 消费者重复消费：
由于⽹络抖动，消费者没有返回ack给broker，导致消费者重试消费。


如何保证幂等性消费？

mysql 插⼊业务id作为主键，主键是唯⼀的，所以⼀次只能插⼊⼀条

**使⽤redis或zk的分布式+消息的唯一ID**：
对于分布式锁仅仅保证该消费流程不会产生并发问题，即可能由于消费者没有即使返回ack等原因，导致重新发送消息。
而唯一ID则保证，即便在特殊形况下，同一条消息被发送了多次，但是由于唯一ID只会进行一次消费。

实际上讲，唯一ID已经保证不会重复消费，分布式锁则是进一步保证消费流程（分布式消费）的并发问题。



# 数据同步机制

![image.png](http://obimage.wenzhuo4657.cn/20240512183549.png)



# 消息模型

## 消费模式分类
[消费者负载均衡 | RocketMQ](https://rocketmq.apache.org/zh/docs/featureBehavior/08consumerloadbalance)
根据消费者组的关系
- 组间广播消费：每个组内只有一个消费者实例
- 组内共享消费：组内有多个消费者。
![](img/Pasted%20image%2020250209155319.png)
当然，这并没有改变组间广播消费，对于每个组的消费进度，依旧是互不相关的，这里只是提出一种实现分组方式来实现广播消费。


消费模式/消息模型：
通过消费者属性进行设置，二者的区别在于消费进度维护在什么地方？
MessageModel.CLUSTERING（集群模式）：消费进度维护在broker中，组内消费者共同管理。
MessageModel.BROADCASTING（广播模式）：消费进度维护在消费者本地实例当中独自管理。


## 消费者分类

消费者的三个节点：拉取消息、处理消息、返回结果。

### PushConsumer

消费者的三个节点都通过预定义的方式自动处理，实际上由程序员处理的仅仅是处理消息，但被设置在边界----监听函数当中。

在消费者实例中设置监听器函数，通过返回值来判断消费是否成功。
```
5.2SDK
consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
@Override  
public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);  
return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
}  
});  
consumer.start();
```

上述用于表示返回值的菜单类
```
public enum ConsumeConcurrentlyStatus {  
/**  
* Success consumption  
*/  
CONSUME_SUCCESS,  
/**  
* Failure consumption,later try to consume  
*/  
RECONSUME_LATER;  
}
```

1,必须同步处理：由于要求监听器函数严格按照返回值判断是否消费成功，
2，轮询执行：这意味着消息的响应速度是均匀的。
在PushConsumer类型中，消息的实时处理能力是基于SDK内部的典型Reactor线程模型实现的。如下图所示，SDK内置了一个长轮询线程，先将消息异步拉取到SDK内置的缓存队列中，再分别提交到消费线程中，触发监听器执行本地消费逻辑。
![](img/Pasted%20image%2020250209161721.png)




###  SimpleConsumer

[消费者分类 | RocketMQ](https://rocketmq.apache.org/zh/docs/featureBehavior/06consumertype#simpleconsumer)
拉取消息、消费、提交三个控制环节全都交由代码执行，适合于需要高度自定义消费流程的场景。

### PullConsumer

主动拉取，但是该模式和push模式一样，都是基于客户端缓存，只是对于push而言，他是由客户端主动推动给消费者实例，而pull则是由消费者主动从客户端缓存中拉取。


## 消息的负载均衡
5.0 SDK下，PushConsumer默认使用消息粒度负载均衡，对于3.x/4.x等Remoting协议SDK 仍然使用了队列粒度负载均衡。


### 消息颗粒度

消息粒度负载均衡策略中，同一消费者分组内的多个消费者将按照消息粒度平均分摊主题中的所有消息，即同一个队列中的消息，可被平均分配给多个消费者共同消费。




### 队列粒度负载均衡









# 零拷贝

[面试系列之-rocketmq零拷贝原理-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2203072)

DMA：帮助cpu执行重复的数据拷贝工作
内核缓冲区（page buff等机制） ：提高文件读取速度，访问文件并不会直接访问磁盘
sockets缓冲区：发送或者待发送的(网络请求)数据准备区域
网卡：网卡则直接用于发送数据。
用户缓存：该区域大部分程序都有，主要为了避免并发问题，而在自己的工作缓存区域设立。

以上缓存区域大多为了提高全局的访问速度，但是对于rocketmq这样的消息发送组件来说则显得鸡肋，主要原因
1，mq不需要修改信息，仅仅进行转发，所以用户缓存和内核缓存功能重复了。
2，对于高并发场景来说，我们更期望直接将数据从磁盘转移到网卡中发送，并不需要sockets缓冲区，当然如果是数据过多，还是需要等待发送完成的。

*零拷贝技术的实质，是提供系统函数突破缓存机制，而并非应用层编码，换句话说，这是系统为了方便用户给出的更高权限的函数*
## mmap+write（rocketmq）
![](img/Pasted%20image%2020250205125259.png)



用户缓冲区和页缓存共享，这样做可以避免一次拷贝，但仍需两次系统调用，4次内核切换。


## 扩展
### sendfile（kafka使用sendfile）

页缓存和socket缓冲区直接传输，一次系统调用两次内核切换
![](img/Pasted%20image%2020250205125851.png)

由于cpu拷贝（页缓存-》socket缓冲区）在内核完成，属于系统调用sendfile，该过程应用程序无法管理，所以对于sendfile零拷贝来说，他只支持bio，单线程阻塞操作。
而mmap+write由于共享了缓冲区，应用程序可以有直接看到缓存数据，所以可以进行nio。

### 网卡支持SG-DMA技术的sendfile


一次系统调用，两次内核切换。
![](img/Pasted%20image%2020250205130010.png)
