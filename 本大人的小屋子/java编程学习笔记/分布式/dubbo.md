早期是基于java高性能的一款轻量级的rpc框架，目前已经扩展为一套易用、高性能的web、rpc框架，用于微服务领域。

阿里提出的微服务解决方案，DNS（D：dubbo N:nacos S:sentinel）,在阿里云内部已经落地、



rpc架构：面向进程的架构

1，由垂直架构（即前后端分离 web系统和后台系统部署在不同的tomcat中）演化而来，
2，解决的是子系统之间的调用，

优点
1，子系统可以复用，即公用模块

rpc架构的问题：
1，服务调用出现问题时，无解决方案
2，多个服务调用同一个模块时，该模块会出现单体架构的热点问题，导致部分服务无法访问，且注意，该问题不能通过水平扩展架构解决，因为其调用服务是一个模块，而不是一个进程级的服务，


soa架构：rpc的演化，面相服务的架构
相比于roc架构中，引出了服务（将某个模块单独抽离出来）的概念，将其独立为一个虚拟机进程，并且做水平扩展，
一般而言，会将多个公用的模块抽离出来作为一个服务，对于服务，也需要做类似于集群的管理，这里称之为服务治理，
常见服务治理有：1，注册中心，2，负载均衡3，容错，4，配置中心，5，限流


企业服务总线ESB：
对于服务的复用更加灵活，允许不同编程语言开发的系统调用该服务，起到一个中间服务的作用，将数据转换为一个双方能识别的中间数据类型。

分为：
1，同步调用
2，异步调用，一般将mq作为中间层进行转换，这意味着不需要将数据转换为中间类型，直接进行调用，然后服务方作为mq的生产者输出。


微服务结构：soa架构的升级
消除调用子系统，将全部的模块都抽离为服务，微服务代表框架：springCloud,DNS


# Dubbo3核心
易用性
开箱即用易用性高，如 Java 版本的面向接口代理特性能实现本地透明调用功能丰富，基于原生库或轻量扩展即可实现绝大多数的微服务治理能力。更加完善了多语言支持(GO PYTHON RUST)
超大规模微服务实践
。高性能通信(Triple GRPC)
。高可扩展性(SPI多种序列化方式 多种协议)
。丰富的服务治理能力
。超大规模集群实例水平扩展
云原生有好
容器调度平台(Kubernetes)
将服务的组织与注册交给底层容器平台，如 Kubernetes，这是更云原生的方式。
Service Mesh
原有Mesh结构中通过Sidecar完成负载均衡、路由等操作，但是存在链路的性能损耗大，现有系统迁移繁琐等问题。 Dubbo3
引入Proxyless Mesh，直接和|控制面交互[istio]通信。集成ServiceMesh更为方便，效率更高。







# 基于springboot整合dubbo


## 配置



yml配置：应用名称，私有传输协议的端口，禁止qos启动
![image.png](http://obimage.wenzhuo4657.cn/20240621140717.png)



相当于：
![image.png](http://obimage.wenzhuo4657.cn/20240621140118.png)


发布服务配置：使用注解@DubboService  
```
@Slf4j  
@DubboService  
public class UserServiceImpl implements UserService{  
@Override  
public boolean login(String name, String password) {  
log.info("账号：{}，密码：{}",name,password);  
return false;  
}  
}
```
相当于：
![image.png](http://obimage.wenzhuo4657.cn/20240621140104.png)

、
接收服务：使用注解
![image.png](http://obimage.wenzhuo4657.cn/20240621140903.png)
相当于：
![image.png](http://obimage.wenzhuo4657.cn/20240621140954.png)


##  注解

### @EnableDubbo

provider必须添加，
consumer根据情况必须添加，但无论如何，添加上不会报错，


作用：
1，扫描@DubboService注解，实例化后将其发布到rpc服务。
**扫描范围：与springboot扫描bean的范围相同，可更改**

1，注解属性更改

![image.png](http://obimage.wenzhuo4657.cn/20240621151339.png)


2，yml配置文件更改
![image.png](http://obimage.wenzhuo4657.cn/20240621151217.png)




# dubbo开发

## 直连应用
指consumer直接调用provider，无需注册中心的接入，

核心概念：
provider:服务提供方
consumer:服务调用方
网路通信：consumer向某个服务方传递服务的输入，provider返回服务调用的结果


网络通信中的重要概念：协议 、通信方式、序列化方式

![image.png](http://obimage.wenzhuo4657.cn/20240621161536.png)

这里的协议体现为配置中的
![](http://obimage.wenzhuo4657.cn/20240621161503.png)


服务器（通信方式）：根据协议不同进行选择
传输层协议：bio、nio、netty等
应用层：tomcat、resin、Jetty


dubbo内置的通信方式默认为netty4
![image.png](http://obimage.wenzhuo4657.cn/20240621162606.png)




序列化:
序列化方式的不同，会导致实际传输数据的大小不同，即将原始数据序列化之后的数据。并且序列化、反序列化耗时不同。
默认序列化方式为hassian
![image.png](http://obimage.wenzhuo4657.cn/20240621162520.png)

注意如果更改序列化方式，对应调用服务的url就需要带上对应的参数，
## 序列化

dubbo默认序列化方式：Hassian

常见序列化：
Dubbo:与dubbo协议区分，这是阿里开发的一种高效序列化方式，暂时不推荐在生产环境中使用
Json:fastjson2或者dubbo中简单实现的序列化方式,且注意如果使用json，一般就使用http协议
Kryo:java序列化方式，用于替换Hassian,是一种非常成熟的序列化实现，在多个大型开源项目中使用
FST:java序列化方式，用于替换Hassian，较新的序列化实现，缺少成功的案例。
跨语言的序列化方式：ProtoBuf、Thrift、Avro、MsgPack


![image.png](http://obimage.wenzhuo4657.cn/20240623122403.png)



## protocol（协议）

由于绝大部分产品的流量入口都是http协议，所以选用rpc协议不可避免的一个问题就是，http->rpc协议，对应的架构方式有中心化架构和去中心化架构，如下图
![image.png](http://obimage.wenzhuo4657.cn/20240623130949.png)
![image.png](http://obimage.wenzhuo4657.cn/20240623130958.png)

如图可知，对于中心化架构，实际上是通过一个入口BFF将Http->rpc协议调用对应的服务，对于去中心架构，则是将压力转到网关处，但在实际应用中，可以通过多协议发布绕过协议转换，所以在上图去中心架构图中，api网关向dubbo集群发出的请求依旧是[[http|http]]。

使用不同的 rpc 协议也会影响架构选择，triple 协议由于原生支持 HTTP 访问，因此对两种架构方式都可以无差别支持，并且接入原理上也会更简单直接。而 dubbo 协议作为 Dubbo2 时代主推的协议，由于是基于 tcp 的二进制协议，因此在接入方式上存在一些不同。


### dubbo协议

 dubbo 协议是基于 TCP 的二进制私有协议，因此更适合作为后端微服务间的高效 RPC 通信协议，这也导致 dubbo 协议对于前端流量接入不是很友好，实际应用中有以下两种方式。


- **多协议发布【推荐】** ，为 dubbo 协议服务暴露 rest 风格的 http 协议访问方式，这种模式架构上会变得非常简单，通用的网关产品即可支持。
- - **通过网关实现 http->dubbo 协议转换**，这种方式需要将 http 协议转换为后端服务能识别的 dubbo 协议，要求网关必须支持 dubbo 协议。


多协议发布配置：
```java
dubbo:
  protocol:
    dubbo: dubbo
    port: 20880
    ext-protocol: tri
```



### triple
riple 同时支持跑在 HTTP/1、HTTP/2 上：

- 在后端服务之间使用高效的 triple 二进制协议。
- 对于前端接入层，则支持所有标准 HTTP 工具如 cURL 等以标准 application/json 格式请求后端服务。
使用 triple 协议后，不再需要泛化调用、http -> dubbo 协议转换等步骤，任何主流的网关设备都可以通过 http 流量直接访问后端 triple 协议服务。


在真正的生产环境下，**唯一需要网关解决的只剩下地址发现问题，即如何动态感知后端 triple 服务的实例变化？** 好消息是，目前几款主流的开源网关产品如 Apache APISIX、Higress 等普遍支持以 Nacos、[[../zookeeper|Zookeeper]]、Kubernetes 作为 upstream 数据源。


参考文章：
[阿里云 - Dubbo3 服务原生支持 http 访问，兼具高性能与易用性 - 阿里巴巴云原生 - SegmentFault 思否](https://segmentfault.com/a/1190000044963384)




## Dubbo注册中心
![image.png](http://obimage.wenzhuo4657.cn/20240623135314.png)


注册中心的核心作用：
- 1. 服务注册：服务提供者(Provider)在启动时，会将⾃身可提供的服务注册到注册中⼼

- 2. 服务发现：服务消费者(Consumer)在启动时，会向注册中⼼订阅⾃⼰需要的服务，

- 3. 服务路由：负载均衡、容错

- 4. 服务监控：注册中⼼可以记录服务的调⽤次数、调⽤延迟等信息，对服务的质量进⾏监控



### DubboAdmin





