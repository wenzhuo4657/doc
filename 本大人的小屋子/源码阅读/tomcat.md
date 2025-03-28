
## 架构
[【Spring MVC + Tomcat】追本溯源，Spring MVC是如何和Tomcat关联到一块的呢？ - 酷酷- - 博客园](https://www.cnblogs.com/kukuxjx/p/17325621.html)
[Tomcat总体架构，启动流程与处理请求流程 - Cuzzz - 博客园](https://www.cnblogs.com/cuzzz/p/17381445.html)

[tomcat源码分析01：构建 tomcat9 源码调试环境本文主要记录了在idea下构建tomcat源码环境过程。 1 - 掘金](https://juejin.cn/post/7102599350676619277)


![](img/Pasted%20image%2020250214115023.png)
![](img/Pasted%20image%2020250214115033.png)

一个tomcat一定只有一个server。

## context/Container



![](img/Pasted%20image%2020250214125653.png)

在`tomcat`中，`Context`的实现为`StandardContext`,启动方法为`#startInternal`

###  processServletContainerInitializers


####  查找`ServletContainerInitializer`

ContextConfig#webConfig -》ContextConfig#processServletContainerInitializers
-》 WebappServiceLoader< ServletContainerInitializer>#load(ServletContainerInitializer.class);

```
public class WebappServiceLoader<T> {  
private static final String CLASSES = "/WEB-INF/classes/";  
private static final String LIB = "/WEB-INF/lib/";  
private static final String SERVICES = "META-INF/services/";

}
```

load方法使用上面的成员变量SERVICES进行拼接路径，然后进行java的spi操作获取实例。
```
public List<T> load(Class<T> serviceType) throws IOException {  
String configFile = SERVICES + serviceType.getName();  
  
// Obtain the Container provided service configuration files.  
ClassLoader loader = context.getParentClassLoader();  
Enumeration<URL> containerResources;  
if (loader == null) {  
containerResources = ClassLoader.getSystemResources(configFile);  
} else {  
containerResources = loader.getResources(configFile);  
}  
  
// Extract the Container provided service class names. Each  
// configuration file may list more than one service class name. This  
// uses a LinkedHashSet so if a service class name appears more than  
// once in the configuration files, only the first one found is used.  
//containerServiceClassNames代表从文件中读取的类全限定路径
//containerServiceConfigFiles代表文件路径
LinkedHashSet<String> containerServiceClassNames = new LinkedHashSet<>();  
Set<URL> containerServiceConfigFiles = new HashSet<>();  
while (containerResources.hasMoreElements()) {  
URL containerServiceConfigFile = containerResources.nextElement();  
containerServiceConfigFiles.add(containerServiceConfigFile);  
parseConfigFile(containerServiceClassNames, containerServiceConfigFile);  
}
...........


return loadServices(serviceType, containerServiceClassNames);//内部真正加载实例

}
```

```
WebappServiceLoader< ServletContainerInitializer>#loadServices

List<T> loadServices(Class<T> serviceType, LinkedHashSet<String> servicesFound) throws IOException {  
ClassLoader loader = servletContext.getClassLoader();  
List<T> services = new ArrayList<>(servicesFound.size());  
for (String serviceClass : servicesFound) {  
try {  
Class<?> clazz = Class.forName(serviceClass, true, loader);  
services.add(serviceType.cast(clazz.getConstructor().newInstance()));  
} catch (ReflectiveOperationException | ClassCastException e) {  
throw new IOException(e);  
}  
}  
return Collections.unmodifiableList(services);  
}
```


####  `@HandlesTypes`

processServletContainerInitializers的第一部分使用javaspi获取配置文件中的目标类型实例,第二部分则是通过注解@HandlesTypes操作这些实例，这实际上意味着我们所获取的实例并非单纯的直接应用，而是可能会先更加复杂的应用场景。

并且需要注意的是，该方法的返回值类型是void,这意味着它的作用结果将通过传递的方式传递给其他变量。

观察源码可发现，最终结果在以下两个成员变量当中。
```
/**  
* ServletContainerInitializer 映射到他们表示感兴趣的类。 
*/  
protected final Map<ServletContainerInitializer, Set<Class<?>>> initializerClassMap =  
new LinkedHashMap<>();  
  
/**  
对这些类型感兴趣的 ServletContainerInitializer 的类型映射。
*/  
protected final Map<Class<?>, Set<ServletContainerInitializer>> typeInitializerMap =  
new HashMap<>();
```

其中对于typeInitializerMap的value变量，将会根据注解@HandlesTypes填充

```
Class<?>[] types = ht.value();  
if (types == null) {  
continue;  
}  
  
for (Class<?> type : types) {  
if (type.isAnnotation()) {  
handlesTypesAnnotations = true;  
} else {  
handlesTypesNonAnnotations = true;  
}  
Set<ServletContainerInitializer> scis =  
typeInitializerMap.get(type);  
if (scis == null) {  
scis = new HashSet<>();  
typeInitializerMap.put(type, scis);  
}  
scis.add(sci);  
}
```


关于initializerClassMap的填充，这里猜测是借助typeInitializerMap集合类型，当这个class在initializerClassMap表示key，实例化对象在initializerClassMap表示key时，就进行填充，作为onstaup方法中的Set<Class< ?>> c参数传入。
```
void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;

..........
for (Map.Entry<ServletContainerInitializer, Set<Class<?>>> entry :  
initializers.entrySet()) {  
try {  
entry.getKey().onStartup(entry.getValue(),  
getServletContext());  
} catch (ServletException e) {  
log.error(sm.getString("standardContext.sciFail"), e);  
ok = false;  
break;  
}  
}
```

并在ContenxtConfig中发现关键方法checkHandlesTypes
```
/**  
对于与 Web 应用程序一起打包的类，需要检查类和每个超类是否与 {@link HandlesTypes} 匹配或是否与 {@link HandlesTypes} 匹配。@param javaClass 指定要检查的类 @param javaClassCache 类缓存
*/  
protected void checkHandlesTypes(JavaClass javaClass,  
Map<String,JavaClassCacheEntry> javaClassCache) {
```


对于是spring的SpringServletContainerInitializer就是通过如上形式，会一次性初始化所有的onStartup方法。避免多个META-INF/services加载。
```
// HandlesTypes 指定的类型为 WebApplicationInitializer
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {

    /**
     * webAppInitializerClasses：这里传入的就是 WebApplicationInitializer
     */
    @Override
    public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, 
            ServletContext servletContext) throws ServletException {

        List<WebApplicationInitializer> initializers = new LinkedList<>();

        if (webAppInitializerClasses != null) {
            // 这个for循环里处理了一些操作
            for (Class<?> waiClass : webAppInitializerClasses) {
                if (...) {
                    try {
                        // 反射实例化
                        initializers.add((WebApplicationInitializer)
                                ReflectionUtils.accessibleConstructor(waiClass).newInstance());
                    }
                    ...
                }
            }
        }
        ...
        // 遍历调用 WebApplicationInitializer 的方法
        for (WebApplicationInitializer initializer : initializers) {
            initializer.onStartup(servletContext);
        }
    }

}

```




### 加载数据（Container和web.xml配置）到context中

无论是那种结构，该加载最终都属于context.


1,加载ServletContainerInitializer
```
if (ok) {  
for (Map.Entry<ServletContainerInitializer,  
Set<Class<?>>> entry :  
initializerClassMap.entrySet()) {  
if (entry.getValue().isEmpty()) {  
context.addServletContainerInitializer(  
entry.getKey(), null);  
} else {  
context.addServletContainerInitializer(  
entry.getKey(), entry.getValue());  
}  
}  
}
```

传递到standardContext的成员变量
```
private Map<ServletContainerInitializer,Set<Class<?>>> initializers =  
new LinkedHashMap<>();
```


2,webxml对象保存

configureContext(webXml);





## Connector

这个类没有标准实现也没有子类继承，只有该类，所以只需要查看该类的重要方法即可
![](img/Pasted%20image%2020250215141006.png)

```
Tomcat tomcat = new Tomcat();  
  
// 创建连接器  
Connector connector = new Connector();  
connector.setPort(8080);  
connector.setURIEncoding("UTF-8");  
tomcat.getService().addConnector(connector);  
  
// 创建 contextString docBase = System.getProperty("java.io.tmpdir");  
Context context = tomcat.addContext("", docBase);  
  
// 得到 lifecycleListener 实际类型为 ContextConfigLifecycleListener lifecycleListener = (LifecycleListener)  
Class.forName(tomcat.getHost().getConfigClass())  
.getDeclaredConstructor().newInstance();  
context.addLifecycleListener(lifecycleListener);  
  
tomcat.start();  
tomcat.getServer().await();
```

在上述demo中可以看到，我们手动创建了链接对象并非自动装配，尝试追溯源码后可以发现connextor的三个核心类。


1. `Connector`：连接器
2. `Http11NioProtocol`：http协议的nio实现
3. `NioEndpoint`：NIO量身定制的线程池

![](img/Pasted%20image%2020250215142047.png)
### 初始化`initInternal`

Connector#initInternal -》  ProtocolHandler#init -》  AbstractProtocol#init  -> AbstractEndpoint#init -> AbstractEndpoint#bindWithCleanup -> NioEndpoint#bind()

```
public void bind() throws Exception {  
initServerSocket();  //ServerSocketChannel初始化
  
setStopLatch(new CountDownLatch(1));  
  
// Initialize SSL if needed  
initialiseSsl();  
  
selectorPool.open(getName());  
//1，selector初始化，但并没有绑定socketchannel链接
//2,启动BlockPoller#start
}
```


在blockpoller的run方法中存在SelectionKey变量，这个变量是javanio包中用于感知selector轮询时间状态的一个变量,可以大致认为该线程用于管理selector。

并且该类是nioblockingselector的内部类
![](img/Pasted%20image%2020250215143043.png)

该类和NioEndpoint的关系通过成员变量维系，在NioEndpoint的存在成员变量NioSelectorPool
```
public class NioEndpoint extends AbstractJsseEndpoint<NioChannel,SocketChannel> {  
  

  
// ----------------------------------------------------------------- Fields  
  
private NioSelectorPool selectorPool = new NioSelectorPool();


.........

}
```

总之对于blockpoller的run方法，我们可以笼统的认为是NioEndpoint管理的一部分，用于实现高效的nio链接。

### 启动startInternal

内部源码追溯，发现启动了两个处理nio连接事件的线程。

```
NioEndpoint#startInternal:这里需要注意，在成员装配时，该类就是Connector的默认连接池处理，所以直接寻找到这里也没问题。
@Override  
public void startInternal() throws Exception {  
  
if (!running) {  
running = true;  
paused = false;  
  
if (socketProperties.getProcessorCache() != 0) {  
processorCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,  
socketProperties.getProcessorCache());  
}  
if (socketProperties.getEventCache() != 0) {  
eventCache = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,  
socketProperties.getEventCache());  
}  
if (socketProperties.getBufferPool() != 0) {  
nioChannels = new SynchronizedStack<>(SynchronizedStack.DEFAULT_SIZE,  
socketProperties.getBufferPool());  
}  
  
// Create worker collection  
if (getExecutor() == null) {  
createExecutor();  
}  
  
initializeConnectionLatch();  
  
// Start poller thread  
poller = new Poller();  
Thread pollerThread = new Thread(poller, getName() + "-ClientPoller");  
pollerThread.setPriority(threadPriority);  
pollerThread.setDaemon(true);  
pollerThread.start();  //线程1：处理读写事件
  
startAcceptorThread();  //线程2：处理连接事件
}
```
#### poller.run

观察源码发现，该类和blockpoller一样，都不具备添加连接注册连接selector的功能，他们都仅仅是轮询，然后处理事件。

#### Acceptor.run

这个源码用于处理链接事件。内部存在关键语句。
```
socket = endpoint.serverSocketAccept();
```

追溯可查找到
```
protected SocketChannel serverSocketAccept() throws Exception {  
return serverSock.accept();  
}
```


这里需要注意在nioEndpoint的initServerSocket方法中，默认将ServerSocketChannel设置阻塞模式。


也就是说tomcat获取链接是阻塞的，


### connect究竟如何处理连接事件

nio链接的三大构件，
serversocketchannel：服务端监听
socketchannel:客户端链接
selector：轮询channel，获取目标事件的channel。


对于connect的三个子线程
blockpoller.run
Acceptor.run：接收链接，加入poller的event队列
```
private final SynchronizedQueue<PollerEvent> events =  
new SynchronizedQueue<>();
```



poller.run
events():
将events队列中的事件，注册到成员变量selector中
```
sc.register(getSelector(), SelectionKey.OP_READ, socketWrapper);

public class Poller implements Runnable {  
  
private Selector selector;
}
```


然后在使用selector的API进行查找到有需要处理的channel
```
processKey(sk, socketWrapper);
```

追溯这个方法我们找到一个关键类ocketProcessorBase，他继承了runnable借口，内部存在run方法处理socket。

```
线程池执行或者直接执行
if (dispatch && executor != null) {  
executor.execute(sc);  
} else {  
sc.run();  
}


public abstract class SocketProcessorBase<S> implements Runnable {  
  
protected SocketWrapperBase<S> socketWrapper;  
protected SocketEvent event;  
  
public SocketProcessorBase(SocketWrapperBase<S> socketWrapper, SocketEvent event) {  
reset(socketWrapper, event);  
}  
  
  
public void reset(SocketWrapperBase<S> socketWrapper, SocketEvent event) {  
Objects.requireNonNull(event);  
this.socketWrapper = socketWrapper;  
this.event = event;  
}  
  
  
@Override  
public final void run() {  
synchronized (socketWrapper) {  
// It is possible that processing may be triggered for read and  
// write at the same time. The sync above makes sure that processing  
// does not occur in parallel. The test below ensures that if the  
// first event to be processed results in the socket being closed,  
// the subsequent events are not processed.  
if (socketWrapper.isClosed()) {  
return;  
}  
doRun();  
}  
}  
  
  
protected abstract void doRun();  
}
```

追溯实现到NioEndpoint.SocketProcessor#doRun()

### NioEndpoint.SocketProcessor#doRun()

doRun直接处理请求，其中关键语句为
`state = getHandler().process(socketWrapper, event);`

gethandler返回的AbstractProtocol$ConnectHandler。
自此正是进入传输部分，也就是coyote包。


直接找到Http11Processor类下的service方法，发现关键方法
`getAdapter().service(request, response);`

下一步调用到CoyoteAdapter#service,该类的作用是适配connect,位于connector包下。
![](img/Pasted%20image%2020250216165013.png)


# Adapter


该借口在tomcat中只有一个实现，用于管理connect到container的映射，


注意：
connect并没有直接和contaniner关联，而是通过adapter进行映射，也就是mapper.


# log模块

日志基类： org.apache.juli.logging.Log
日志基类tomcat唯一实现： org.apache.juli.logging.DirectJDKLog
日志工厂： org.apache.juli.logging.LogFactory
日志异常处理： org.apache.juli.logging.LogConfigurationException

定位工厂的唯一无参构造器发现ServiceLoader.load(Log.class)，该类是java spi的工具包用于读取META-INF/services/文件名为接口的全限定类名，并加载实例化的一种机制。

![](img/Pasted%20image%2020250310142514.png)
尝试在源码中添加相关配置之后失败，显示无法实例化，追溯源码后发现，他内部默认使用`c.newInstance()`,也就是空参构造器实例化。


追溯DirectJDKLog唯一的构造器找到LogFactory#getInstance(String name);
```
public Log getInstance(String name) throws LogConfigurationException {  
if (discoveredLogConstructor == null) {  
return DirectJDKLog.getInstance(name);  
}  
  
try {  
return discoveredLogConstructor.newInstance(name);  
} catch (ReflectiveOperationException | IllegalArgumentException e) {  
throw new LogConfigurationException(e);  
}  
}
```


所以对于tomcat来说DirectJDKLog是默认实现，否则就使用资源目录的spi文件中的服务实现。并且同样要求该实现又一个String参数的构造器。

并且注意LogFactory的成员变量
`private static final LogFactory singleton = new LogFactory();`


但是在调试中文乱码时会发现他默认使用的是jdk下的日志（也就是JUL：java.util.logging），他们二者之间的关系是？

[Day698.Tomcat的日志框架及实战 -深入拆解 Tomcat & Jetty\_directjdklog-CSDN博客](https://blog.csdn.net/qq_43284469/article/details/126166994)
默认情况下，Tomcat 的日志模板叫作 JULI，JULI 的日志门面采用了 JCL，而具体实现是基于 Java 默认的日志框架 Java Util Logging（JUL）。


日志门面： 充当应用程序和日志框架之间的沟通媒介，可以在程序无感的条件下更换日志框架，


常见的日志实现：JUL、log4j、logback、log4j2
常见的日志门面 ：JCL、slf4j


因此对于tomcat来说，juli是日志门面，他会动态的选择日志实现，源码crtl+F搜索java.util.logging之后找到方法release，停止继续深究。
```
public static void release(ClassLoader classLoader) {  
// JULI's log manager looks at the current classLoader so there is no  
// need to use the passed in classLoader, the default implementation  
// does not so calling reset in that case will break things  
if (!LogManager.getLogManager().getClass().getName().equals(  
"java.util.logging.LogManager")) {  
LogManager.getLogManager().reset();  
}  
}
```



