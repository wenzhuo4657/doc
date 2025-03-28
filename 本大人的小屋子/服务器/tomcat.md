


相关文章及知识：
[Http请求和request、reponse的关系][https://www.cnblogs.com/jokes/p/12523430.html]

Http请求是应用层协议，下层协议是基于TCP协议进行网络通信，对与Http1.0而言，所建立的tcp连接的生命周期是一次请求，所以我们说Http1.0是无连接的，但这不意味着tcp也是无连接的，而Htpp1.1同样基于TCP协议进行网络传输，但它所建立的tcp连接是持久的，所以我们说Http1.1协议是可持续连接的。

而所谓的soket通信是基于TCP/IP协议进行网络通信的，他本身并不是任何任何协议，仅仅是一个api调用，他与http协议没有直接关系，但是却可以根据soketAPI来模拟http协议，


基于计算机网络的知识，我们可以知道无论应用层协议如何封装，都是根据下层协议进行传输数据到达位于不同主机、进程的两个应用，实际上我们的问题是，http协议如何被封装成ip数据包进行传输？到达目标位置时又如何解析为request、reponse？


# 核心概念
![image.png](http://obimage.wenzhuo4657.cn/20240622131833.png)


tomcat要实现的2个核心功能，
- **处理 Socket 连接**，负责网络字节流与 Request 和 Response 对象的转化。
- **加载和管理 Servlet**，以及**处理具体的 Request 请求**。


Tomcat 设计了两个核心组件：

- **连接器（Connector）**：负责和外部通信
- **容器（Container）**：负责内部处理
- **Service**： 连接器和容器组合，**一个 Service 有多个 Connector 和 Container**





## Connector（连接器）

连接器对 Servlet 容器屏蔽了协议及 I/O 模型等的区别，无论是 HTTP 还是 AJP，在容器中获取到的都是一个标准的 ServletRequest 对象。

主要功能以及对应实现的组件：
- **网络通信**：EndPoint：socket通信，根据Tcp/ip协议发送和接收数据包，并将其转化为对应的应用层协议
- **应用层协议解析**：Proccesor：封装response为应用层协议数据包，或者解析http请求数据包为request
- **Tomcat Request/Response 与 ServletRequest/ServletResponse 的转化**：**`Adapter`**:调用对应的容器的对应 Service 方法


![image.png](http://obimage.wenzhuo4657.cn/20240622143639.png)


注意此处的入口EndPoint使用的是socketAPI，这意味着器通信是ip:端口指定目标地址，而非其他模糊的形式，也就是说这对应这某个程序服务。




##  容器（Container）

Tomcat 设计了 4 种容器，分别是 Engine、Host、Context 和 Wrapper。

- **Engine** - Servlet 的顶层容器，包含一 个或多个 Host 子容器；
- **Host** - 虚拟主机，负责 web 应用的部署和 Context 的创建；
- **Context** - Web 应用上下文，包含多个 Wrapper，负责 web 配置的解析、管理所有的 Web 资源；
- **Wrapper** - 最底层的容器，是对 Servlet 的封装，负责 Servlet 实例的创 建、执行和销毁。




## 请求分发servlet的过程
使用组件 Mapper来找到ServletRequest对应的方法，需要注意的`Mapper`组件自身并不直接获取请求路径；相反，它是在请求路径已经被`Connector`和`CoyoteAdapter`解析之后，由`Host`和`Context`容器调用来确定请求应该如何被映射到具体的servlet上。


![image.png](http://obimage.wenzhuo4657.cn/20240622151257.png)




## springBoot+tomcat






