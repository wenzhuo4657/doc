
# 概念

[RocketMQ源码系列(一)：阅读源码前的准备1、获取RocketMQ源码 由于从GitHub上clone代码太慢，这 - 掘金](https://juejin.cn/post/7077079471542665252)


源码`git clone https://gitee.com/apache/rocketmq.git`


设计理念
RocketMQ设计基于主题的发布与订阅模式，其核心功能包括消息 发送、消息存储和消息消费，整体设计追求简单和性能高效，主要体 现在如下3个方面。

- NameServer的设计极其简单，摒弃了业界常用的将 ZooKeeper作为信息管理的“注册中心”，而是*自研NameServer实现元 数据的管理*（topic路由信息等）。从实际需求出发，topic路由信息 无须在集群之间保持强一致，而是追求最终一致性，并且能容忍分钟 级的不一致。正是基于这种特性，RocketMQ的NameServer集群之间互 不通信，这样极大地降低了NameServer实现的复杂度，对网络的要求 也降低了不少，性能相比较ZooKeeper还有了极大的提升。

- *高效的I/O存储机制*。RocketMQ追求消息发送的高吞吐量， RocketMQ的消息存储文件被设计成文件组的概念，组内单个文件大小 固定，方便引入内存映射机制，所有主题的消息存储按顺序编写，极 大地提升了消息的写性能。同时为了兼顾消息消费与消息查找，引入了消息消费队列文件与索引文件。

- *容忍存在设计缺陷*，适当将某些工作下放给RocketMQ使用者。消息中间件的实现者经常会遇到一个难题：如何保证消息一定能 被消息消费者消费，并且只消费一次？

RocketMQ的设计者给出的解决办法是不解决这个难题，而是退而 求其次，只保证消息被消费者消费，**在设计上允许消息被重复消费**。 这样极大地简化了消息中间件的内核，使得实现消息发送高可用变得 非常简单和高效，**消息重复问题由消费者在消息消费时实现幂等**。


 
 nameserver的路由功能
被动通知：namerserver通过心跳机制检测broker是否存活，当检测到节点失联后，只会将其从本地路由表中移除，并没有主动通知生产者。
（*体现了容忍缺陷的设计，即容忍生产者发送失败，需要消息发送端提供容错机制来保证消息发送的高可用性，同时减少mq系统的实现复杂度。*）



> [!NOTE] Title
> 消息客户端与NameServer、Broker的交互
 >Broker每隔30s向NameServer集群的每一台机器发送心跳包， 包含自身创建的topic路由等信息。
消息客户端每隔30s向NameServer更新对应topic的路由信息。
NameServer收到Broker发送的心跳包时会记录时间戳。
NameServer每隔5s会扫描一次brokerLiveTable（存放心跳包的时间戳信息），如果在120s内没有收到心跳包，则认为Broker失效，更新topic的路由信息，将失效的Broker信息移除。





# namesrv的钩子函数
org.apache.rocketmq.namesrv中的start方法
```
public static NamesrvController start(final NamesrvController controller) throws Exception {  
  
if (null == controller) {  
throw new IllegalArgumentException("NamesrvController is null");  
}  
  
boolean initResult = controller.initialize();  //初始化的实际操作
if (!initResult) {  
controller.shutdown();  
System.exit(-3);  
}  
  
Runtime.getRuntime().addShutdownHook(new ShutdownHookThread(log, (Callable<Void>) () -> {  
controller.shutdown();  
return null;  
}));  //定义钩子函数，用于关闭资源
  
controller.start();  
  
return controller;  
}

```




# 路由注册

虽说是路由注册，但实际上包含了topic和心跳信息，内部更是作为定时任务执行。

## broker的路由注册：发送心跳和topic信息

BrokerStartup#start() -> BrokerController#start()  方法内部会有一个定时任务
```
scheduledFutures.add(this.scheduledExecutorService.scheduleAtFixedRate(new AbstractBrokerRunnable(this.getBrokerIdentity()) {  
@Override  
public void run0() {  
try {  
if (System.currentTimeMillis() < shouldStartTime) {  
BrokerController.LOG.info("Register to namesrv after {}", shouldStartTime);  
return;  
}  
if (isIsolated) {  
BrokerController.LOG.info("Skip register for broker is isolated");  
return;  
}  
BrokerController.this.registerBrokerAll(true, false, brokerConfig.isForceRegister());  //注册broker
} catch (Throwable e) {  
BrokerController.LOG.error("registerBrokerAll Exception", e);  
}  
}  
}, 1000 * 10, Math.max(10000, Math.min(brokerConfig.getRegisterNameServerPeriod(), 60000)), TimeUnit.MILLISECONDS));

```


`BrokerController.this.registerBrokerAll(true, false, brokerConfig.isForceRegister());`
这里需要区分this的指向，在该语句中，通过new AbstractBrokerRunnable传递一个匿名内部类的实例，而this的指向也是该匿名内部类的实例，因此BrokerController.this是作为外部类实例和匿名内部实例的this进行区分而产生的写法。两者的作用域也不同。
[Java-“this”和“类名.this”以及“类名.class”的区分和详解-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1585854)



进一步追溯之后也能够看到更多细节，

1，传递的真正实体为TopicConfig类,并且是使用复制快照的方式保证了并发安全。
```
ConcurrentMap<String, TopicConfig> topicConfigMap = this.getTopicConfigManager().getTopicConfigTable();  
ConcurrentHashMap<String, TopicConfig> topicConfigTable = new ConcurrentHashMap<>();  
  
for (TopicConfig topicConfig : topicConfigMap.values()) {  
//内容复制到  topicConfigTable中
}
```

2，发送的数据格式为二进制
```
requestBody.setTopicConfigSerializeWrapper(TopicConfigAndMappingSerializeWrapper.from(topicConfigWrapper));//topicConfigWrapper是对topicConfigTable的进一步封装
。。。。。
final byte[] body = requestBody.encode(compressed);
。。。。。。
RegisterBrokerResult result = registerBroker(namesrvAddr, oneway, timeoutMills, requestHeader, body);
```


## namesrv接收：更新broker

DefaultRequestProcessor#processRequest:默认的请求处理，内部会通过请求code来判断处理器方法
```
switch (request.getCode()) {
.....
case RequestCode.REGISTER_BROKER:  
return this.registerBroker(ctx, request);
.......
}
```


DefaultRequestProcessor#registerBroker -> this.namesrvController.getRouteInfoManager().registerBroker():注册并更新存货信息等
```
this.lock.writeLock().lockInterruptibly();//aqs可重入锁，非公平锁
。。。。
BrokerData brokerData = this.brokerAddrTable.get(brokerName);  //路由注册
if (null == brokerData) {  
registerFirst = true;  
brokerData = new BrokerData(clusterName, brokerName, new HashMap<>());  
this.brokerAddrTable.put(brokerName, brokerData);  
}
。。。。。
//填充路由等操作
。。。。
BrokerAddrInfo brokerAddrInfo = new BrokerAddrInfo(clusterName, brokerAddr);  
BrokerLiveInfo prevBrokerLiveInfo = this.brokerLiveTable.put(brokerAddrInfo,  
new BrokerLiveInfo(  
System.currentTimeMillis(),  
timeoutMillis == null ? DEFAULT_BROKER_CHANNEL_EXPIRED_TIME : timeoutMillis,  
topicConfigWrapper == null ? new DataVersion() : topicConfigWrapper.getDataVersion(),  
channel,  
haServerAddr));//broker存活信息更新
。。。。。。
if (filterServerList != null) { //消息过滤填充，
if (filterServerList.isEmpty()) {  
this.filterServerTable.remove(brokerAddrInfo);  
} else {  
this.filterServerTable.put(brokerAddrInfo, filterServerList);  
}  
}  
  
if (MixAll.MASTER_ID != brokerId) {  
String masterAddr = brokerData.getBrokerAddrs().get(MixAll.MASTER_ID);  
if (masterAddr != null) {  
BrokerAddrInfo masterAddrInfo = new BrokerAddrInfo(clusterName, masterAddr);  
BrokerLiveInfo masterLiveInfo = this.brokerLiveTable.get(masterAddrInfo);  
if (masterLiveInfo != null) {  
result.setHaServerAddr(masterLiveInfo.getHaServerAddr());  
result.setMasterAddr(masterAddr);  
}  
}  
}
。。。。。。
```



NameServer与Broker保持长连接，Broker的状态信息存储 在brokerLive-Table中，NameServer每收到一个心跳包，将更新 brokerLiveTable中关于Broker的状态信息以及路由表 （topicQueueTable、brokerAddrTable、brokerLiveTable、 filterServer-Table）。
**更新上述路由表（HashTable）使用了锁粒度 较少的读写锁**，允许多个消息发送者并发读操作，保证消息发送时的 高并发。同一时刻NameServer只处理一个Broker心跳包，多个心跳包 请求串行执行。这也是读写锁经典的使用场景。



# 路由删除

在namesrv中的初始化任务中存在
```
public boolean initialize() {  
loadConfig();  
initiateNetworkComponents();  
initiateThreadExecutors();  
registerProcessor();  
startScheduleService();  //调用
initiateSslContext();  
initiateRpcHooks();  
return true;  
}
```

NamesrvStartup#start -> NamesrvController#initialize() -> NamesrvController#startScheduleService()  -> NamesrvController.this.routeInfoManager::scanNotActiveBroker()

```
public void scanNotActiveBroker() {  
try {  
log.info("start scanNotActiveBroker");  
for (Entry<BrokerAddrInfo, BrokerLiveInfo> next : this.brokerLiveTable.entrySet()) {  
long last = next.getValue().getLastUpdateTimestamp();  
long timeoutMillis = next.getValue().getHeartbeatTimeoutMillis();  
if ((last + timeoutMillis) < System.currentTimeMillis()) {  
RemotingHelper.closeChannel(next.getValue().getChannel());  
log.warn("The broker channel expired, {} {}ms", next.getKey(), timeoutMillis);  
this.onChannelDestroy(next.getKey());  
}  
}  
} catch (Exception e) {  
log.error("scanNotActiveBroker exception", e);  
}  
}
```


时间戳计算判断是否broker过期并删除。

# 路由拉取(namesrv视角)

[RocketMQ源码阅读 - yudoge - 博客园](https://www.cnblogs.com/lilpig/p/17627891.html)

这篇博客给了我一个提示，就是查看父类以及处理器划分，在源码中可以看到，namesrv模块下的processor包下的所有类都实现了NettyRequestProcessor，这是他们的共同点。
```
public interface NettyRequestProcessor {  
RemotingCommand processRequest(ChannelHandlerContext ctx, RemotingCommand request)  
throws Exception;  
  
boolean rejectRequest();  
}
```

RemotingCommand、NettyRequestProcessor都是属于remoting模块下的实现类，因此这里我们对它保持一个可以完成网络传输的印象就可以了。


而在我们所关注的namesrv模块中，有三个这样的处理器实现。
![](img/Pasted%20image%2020250206101049.png)

## clientRequestProcessor

```
@Override  
public RemotingCommand processRequest(final ChannelHandlerContext ctx,  
final RemotingCommand request) throws Exception {  
return this.getRouteInfoByTopic(ctx, request);  
}
```

netty网络传输处理器方法中可以看到关键方法this.getRouteInfoByTopic(ctx, request); 

```
public RemotingCommand getRouteInfoByTopic(ChannelHandlerContext ctx,  
RemotingCommand request) throws RemotingCommandException {  
（1）响应准备
final RemotingCommand response = RemotingCommand.createResponseCommand(null); //响应类定义 
final GetRouteInfoRequestHeader requestHeader =  
(GetRouteInfoRequestHeader) request.decodeCommandCustomHeader(GetRouteInfoRequestHeader.class);//解码数据，  
  
boolean namesrvReady = needCheckNamesrvReady.get() && System.currentTimeMillis() - startupTimeMillis >= TimeUnit.SECONDS.toMillis(namesrvController.getNamesrvConfig().getWaitSecondsForService());  //namesrvReady是否可以读的检查
  
if (namesrvController.getNamesrvConfig().isNeedWaitForService() && !namesrvReady) {  
log.warn("name server not ready. request code {} ", request.getCode());  
response.setCode(ResponseCode.SYSTEM_ERROR);  
response.setRemark("name server not ready");  
return response;  
}  

（2）响应中：准备响应数据，响应包装等
TopicRouteData topicRouteData = this.namesrvController.getRouteInfoManager().pickupTopicRouteData(requestHeader.getTopic());  
  
if (topicRouteData != null) {  
//topic route info register success ,so disable namesrvReady check  
if (needCheckNamesrvReady.get()) {  
needCheckNamesrvReady.set(false);  //关闭响应检查
}  
  
if (this.namesrvController.getNamesrvConfig().isOrderMessageEnable()) {  
String orderTopicConf =  
this.namesrvController.getKvConfigManager().getKVConfig(NamesrvUtil.NAMESPACE_ORDER_TOPIC_CONFIG,  
requestHeader.getTopic());//kvconfig读取，namesrv级别 
topicRouteData.setOrderTopicConf(orderTopicConf);  
}  
  
byte[] content;  
Boolean standardJsonOnly = Optional.ofNullable(requestHeader.getAcceptStandardJsonOnly()).orElse(false);  
if (request.getVersion() >= MQVersion.Version.V4_9_4.ordinal() || standardJsonOnly) {  
content = topicRouteData.encode(SerializerFeature.BrowserCompatible,  
SerializerFeature.QuoteFieldNames, SerializerFeature.SkipTransientField,  
SerializerFeature.MapSortField);  
} else {  
content = topicRouteData.encode();  
}  
  
response.setBody(content);  
response.setCode(ResponseCode.SUCCESS);  
response.setRemark(null);  
return response;  
}  
  
response.setCode(ResponseCode.TOPIC_NOT_EXIST);  
response.setRemark("No topic route info in name server for the topic: " + requestHeader.getTopic()  
+ FAQUrl.suggestTodo(FAQUrl.APPLY_TOPIC_URL));  
return response;  
}
```

从源码中可以观察到
1，namesrcReady可读检查，并且在成员变量中定义了默认值为true开启。并且该判断是纯时间判断
`private AtomicBoolean needCheckNamesrvReady = new AtomicBoolean(true);`
`boolean namesrvReady = needCheckNamesrvReady.get() && System.currentTimeMillis() - startupTimeMillis >= TimeUnit.SECONDS.toMillis(namesrvController.getNamesrvConfig().getWaitSecondsForService()); `

疑问：这里似乎过长？由于是45分钟，所以ClientRequestProcessor对象在创建之后的45分钟内都没办法使用？
2，kvconfig读取，namesrv级别的配置

`private String kvConfigPath = System.getProperty("user.home") + File.separator + "namesrv" + File.separator + "kvConfig.json";`
例如：`C:\Users\14783\namesrv\kvConfig.json`

并且该配置在namesrv启动时读取，定义在模板方法
```
public boolean initialize() {  
loadConfig();  
initiateNetworkComponents();  
initiateThreadExecutors();  
registerProcessor();  
startScheduleService();  
initiateSslContext();  
initiateRpcHooks();  
return true;  
}
```

追溯loadconfig可以找到关键代码语句。
NamesrvController#initialize -》  NamesrvController#loadConfig  - 》 KVConfigManager#load()

```
public void load() {  
String content = null;  
try {  
content = MixAll.file2String(this.namesrvController.getNamesrvConfig().getKvConfigPath());  
.........
}
```


并且在MixAll.file2String会做文件是否可用的检查。

## DefaultRequestProcessor

DefaultRequestProcessor和clientRequestProcessor，他的网络传输拥有处理更多的请求，内部通过switch选择。
```
@Override  
public RemotingCommand processRequest(ChannelHandlerContext ctx,  
RemotingCommand request) throws RemotingCommandException {  
  
if (ctx != null) {  
log.debug("receive request, {} {} {}",  
request.getCode(),  
RemotingHelper.parseChannelRemoteAddr(ctx.channel()),  
request);  
}  
  
switch (request.getCode()) {  
case RequestCode.PUT_KV_CONFIG:  
return this.putKVConfig(ctx, request);  
case RequestCode.GET_KV_CONFIG:  
return this.getKVConfig(ctx, request);  
case RequestCode.DELETE_KV_CONFIG:  
return this.deleteKVConfig(ctx, request);  
case RequestCode.QUERY_DATA_VERSION:  
return this.queryBrokerTopicConfig(ctx, request);  
case RequestCode.REGISTER_BROKER:  
return this.registerBroker(ctx, request);  
case RequestCode.UNREGISTER_BROKER:  
return this.unregisterBroker(ctx, request);  
case RequestCode.BROKER_HEARTBEAT:  
return this.brokerHeartbeat(ctx, request);  
case RequestCode.GET_BROKER_MEMBER_GROUP:  
return this.getBrokerMemberGroup(ctx, request);  
case RequestCode.GET_BROKER_CLUSTER_INFO:  
return this.getBrokerClusterInfo(ctx, request);  
case RequestCode.WIPE_WRITE_PERM_OF_BROKER:  
return this.wipeWritePermOfBroker(ctx, request);  
case RequestCode.ADD_WRITE_PERM_OF_BROKER:  
return this.addWritePermOfBroker(ctx, request);  
case RequestCode.GET_ALL_TOPIC_LIST_FROM_NAMESERVER:  
return this.getAllTopicListFromNameserver(ctx, request);  
case RequestCode.DELETE_TOPIC_IN_NAMESRV:  
return this.deleteTopicInNamesrv(ctx, request);  
case RequestCode.REGISTER_TOPIC_IN_NAMESRV:  
return this.registerTopicToNamesrv(ctx, request);  
case RequestCode.GET_KVLIST_BY_NAMESPACE:  
return this.getKVListByNamespace(ctx, request);  
case RequestCode.GET_TOPICS_BY_CLUSTER:  
return this.getTopicsByCluster(ctx, request);  
case RequestCode.GET_SYSTEM_TOPIC_LIST_FROM_NS:  
return this.getSystemTopicListFromNs(ctx, request);  
case RequestCode.GET_UNIT_TOPIC_LIST:  
return this.getUnitTopicList(ctx, request);  
case RequestCode.GET_HAS_UNIT_SUB_TOPIC_LIST:  
return this.getHasUnitSubTopicList(ctx, request);  
case RequestCode.GET_HAS_UNIT_SUB_UNUNIT_TOPIC_LIST:  
return this.getHasUnitSubUnUnitTopicList(ctx, request);  
case RequestCode.UPDATE_NAMESRV_CONFIG:  
return this.updateConfig(ctx, request);  
case RequestCode.GET_NAMESRV_CONFIG:  
return this.getConfig(ctx, request);  
default:  
String error = " request type " + request.getCode() + " not supported";  
return RemotingCommand.createResponseCommand(RemotingSysResponseCode.REQUEST_CODE_NOT_SUPPORTED, error);  
}  
}
```


## ClusterTestRequestProcessor
该处理器继承了clientRequestProcessor，并重写了getRouteInfoByTopic方法，此外没有其他重要的实现。并且注意，对于重写的getRouteInfoByTopic没有使用namesrcReady检查，（不是很理解）


# namesrv请求处理器注册

```
public boolean initialize() {  
loadConfig();  
initiateNetworkComponents();  
initiateThreadExecutors();  
registerProcessor();  //处理器注册
startScheduleService();  
initiateSslContext();  
initiateRpcHooks();  
return true;  
}
```

```
private void registerProcessor() {  
if (namesrvConfig.isClusterTest()) {  
  
this.remotingServer.registerDefaultProcessor(new ClusterTestRequestProcessor(this, namesrvConfig.getProductEnvName()), this.defaultExecutor);  
} else {  
// Support get route info only temporarily  
ClientRequestProcessor clientRequestProcessor = new ClientRequestProcessor(this);  
this.remotingServer.registerProcessor(RequestCode.GET_ROUTEINFO_BY_TOPIC, clientRequestProcessor, this.clientRequestExecutor);  
  
this.remotingServer.registerDefaultProcessor(new DefaultRequestProcessor(this), this.defaultExecutor);  
}  
}
```


同时这里也可以看到clientrequestProessor的请求代码`RequestCode.GET_ROUTEINFO_BY_TOPIC`



# 路由拉取(客户端视角：生产者、消费者)
## 生产者


### 启动
默认实现的生产者实例安装

DefaultMQProducer#start() -> DefaultMQProducerImpl#start() -> MQClientManager.getInstance().getOrCreateMQClientInstance(this.defaultMQProducer, rpcHoo)  -> MQClientInstance # new MQClientInstance(ClientConfig clientConfig, int instanceIndex, String clientId, RPCHook rpcHook)

该实例安装内部有一句非常重要的代码
```
this.mQClientAPIImpl = new MQClientAPIImpl(this.nettyClientConfig, clientRemotingProcessor, rpcHook, clientConfig, channelEventListener);
```

观察参数可以发现，内部存在
clientRemotingProcessor：客户端远程处理器
rpcHook：大概是钩子函数之类的，用于控制、增强clientRemotingProcessor
channelEventListener：监听消息传输等，默认实现中会存在一个与broker建立连接的实现。
```
@Override  
public void onChannelActive(String remoteAddr, Channel channel) {  
for (Map.Entry<String, HashMap<Long, String>> addressEntry : brokerAddrTable.entrySet()) {  
for (Map.Entry<Long, String> entry : addressEntry.getValue().entrySet()) {  
String addr = entry.getValue();  
if (addr.equals(remoteAddr)) {  
long id = entry.getKey();  
String brokerName = addressEntry.getKey();  
if (sendHeartbeatToBroker(id, brokerName, addr, false)) {  
rebalanceImmediately();  
}  
break;  
}  
}  
}  
}
```

追溯调用链路可发现，该listen在netty连接的事件中被大量调用。
```
NettyRemotingAbstract#run()
@Override  
public void run() {  
log.info(this.getServiceName() + " service started");  
  
final ChannelEventListener listener = NettyRemotingAbstract.this.getChannelEventListener();  
  
while (!this.isStopped()) {  
try {  
NettyEvent event = this.eventQueue.poll(3000, TimeUnit.MILLISECONDS);  
if (event != null && listener != null) {  
switch (event.getType()) {  
case IDLE:  
listener.onChannelIdle(event.getRemoteAddr(), event.getChannel());  
break;  
case CLOSE:  
listener.onChannelClose(event.getRemoteAddr(), event.getChannel());  
break;  
case CONNECT:  
listener.onChannelConnect(event.getRemoteAddr(), event.getChannel());  
break;  
case EXCEPTION:  
listener.onChannelException(event.getRemoteAddr(), event.getChannel());  
break;  
case ACTIVE:  
listener.onChannelActive(event.getRemoteAddr(), event.getChannel());  
break;  
default:  
break;  
  
}  
}  
} catch (Exception e) {  
log.warn(this.getServiceName() + " service has exception. ", e);  
}  
}  
  
log.info(this.getServiceName() + " service end");  
}
```


### 发送

在启动生产者源码汇总，仅仅看到一个在listen中指向broker列表并在事件中建立连接的函数
```
private final ConcurrentMap<String, HashMap<Long, String>> brokerAddrTable = MQClientInstance.this.brokerAddrTable;
```

但是这里需要注意的是，这个成员变量brokerAddrTable在初始化时默认值就已经被初始化了。
`private final ConcurrentMap<String, HashMap<Long, String>> brokerAddrTable = new ConcurrentHashMap<>();`

因此后续应该时通过维护这个变量来建立连接，那么究竟在什么时候拉取呢？

追溯这个成员变量的调用找到MQClientInstance#updateTopicRouteInfoFromNameServer方法,并且似乎仅仅有这一个地方传入。
```
public boolean updateTopicRouteInfoFromNameServer(final String topic, boolean isDefault,  
DefaultMQProducer defaultMQProducer) {
........
this.topicRouteTable.put(topic, cloneTopicRouteData);
.........
}
```


存在关键拉去方法
`topicRouteData = this.mQClientAPIImpl.getDefaultTopicRouteInfoFromNameServer(clientConfig.getMqClientApiTimeout());`

方法内部追溯到了请求代码`RequestCode.GET_ROUTEINFO_BY_TOPIC`,也就是namesrv中的clientRequestProcessor的请求代码。


追溯发送消息的SendResult sendResult = producer.send(msg, 20 * 1000);可以看到如下判断
```
private TopicPublishInfo tryToFindTopicPublishInfo(final String topic) {  
TopicPublishInfo topicPublishInfo = this.topicPublishInfoTable.get(topic);  
if (null == topicPublishInfo || !topicPublishInfo.ok()) {  
this.topicPublishInfoTable.putIfAbsent(topic, new TopicPublishInfo());  
this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);//拉取信息
topicPublishInfo = this.topicPublishInfoTable.get(topic);  
}  
  
if (topicPublishInfo.isHaveTopicRouterInfo() || topicPublishInfo.ok()) {  
return topicPublishInfo;  
} else {  
this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic, true, this.defaultMQProducer);  
topicPublishInfo = this.topicPublishInfoTable.get(topic);  
return topicPublishInfo;  
}  
}
```

并且在源码中并未看到主动拉取的方法，例如定时器、线程池任务等，这一点也正好符合rocketmq允许发送失败，他的发送原则是只要存在地址就发送，如果发送失败，则进行更新，他并不会检查当前存在地址是否有效。



## 消费者
消费者由于是主动设置topic，并且同样遵从被动拉取地址，所以直接从存储元数据的列表变量处追溯拉取的代码。

RebalanceImpl的成员变量：
`protected final ConcurrentMap<String /* topic */, SubscriptionData> subscriptionInner =new ConcurrentHashMap<>();`

