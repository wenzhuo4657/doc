

## spring-ai-alibaba-chat-example

## 接口梳理

![[Pasted image 20250315212434.png]]

### chatmodel多实例注入
问题描述，当我新建一个工程将代码直接复制过去时，他并没有报错多实例错误，但是在这个示例的代码包中确实报错了。
![[Pasted image 20250315085129.png]]


深入源码之后发现各个模型的配置类和注入的条件配置方法
![[Pasted image 20250315085829.png]]
![[Pasted image 20250315085858.png]]

可以很清楚的看到他应该时默认注入的，所以在每个应用程序中都应当有多个ChatModel的bean,但为什么最终只有一个呢？


查看自动注入类的注解发现@ConditionalOnClass(MoonshotApi.class)，该注解表明只有当类路径中存在指定类时，才会生效，这也是为什么将配置类默认打开的原因，即便打开了配置类了也会由于没有导入对应的包而使bean注入无效。

需要注意的时，上述配置类均位于`org.springframework.ai.autoconfigure`，而类似于MoonshotApi这样的条件判断类则位于他们各自的依赖当中，如果不导入则不会注入。
![[Pasted image 20250315092945.png]]


所以虽然在依赖中，他们可以检测到对应类，但是在模块执行的类路径当中，对应依赖并不存在，无视报错直接执行即可。




## java.lang.NoClassDefFoundError: com/google/gson/Strictness

示例里有这个，粘贴到我的项目就没有，搞不懂为什么，追源码找到对应依赖加上即可
```
<dependency>  
<groupId>com.google.code.gson</groupId>  
<artifactId>gson</artifactId>  
<version>2.11.0</version>  
</dependency>
```



## autdio

当下的语音模型大多并非是直接进行交互，而是语音-》文本，大模型思考得出结果-》文本-》语音合成。

关键属于有:stt(语音转文本)，tts(文本转语音)



## 自动注入mcp服务端

```
<dependency>  
<groupId>org.springframework.ai</groupId>  
<artifactId>spring-ai-mcp-server-spring-boot-autoconfigure</artifactId>  
<version>1.0.0-M6</version>  
</dependency>
```


方法参数自动注入，为mcp-server自动注入当前spring容器的所有ToolCallbackProvider（也就是工具的回调，可以简单理解为，我们可以通过这个实例索取到他所管理的工具方法）

而同步和异步的处理，他是通过@ConditionalOnProperty判断是否存在指定的配置来选择注入。

```
@Bean  
@ConditionalOnProperty(  
prefix = "spring.ai.mcp.server",  
name = {"type"},  
havingValue = "SYNC",  
matchIfMissing = true  
)  
public McpSyncServer mcpSyncServer(...){}


@Bean  
@ConditionalOnProperty(  
prefix = "spring.ai.mcp.server",  
name = {"type"},  
havingValue = "ASYNC"  
)  
public McpAsyncServer mcpAsyncServer
```

值得注意的是@ConditionalOnProperty的属性havingValue的默认值在（当前版本）是false，所以默认情况下该配置使用同步的mcp服务端。


对于webflux和stdio同理，不过他默认的是stdio.