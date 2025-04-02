
- VectorStore(矢量数据库)：检索数据，用于给ai提供数据。 例如，PgVector，

对于ai来讲，这个数据库的作用仅仅是用于提供数据例如springai所提供的简单实现。[SimpleVectorStore](https://github.com/spring-projects/spring-ai/blob/main/spring-ai-core/src/main/java/org/springframework/ai/vectorstore/SimpleVectorStore.java)




- ai服务提供源：各大厂商的token,或者自建的大模型（可通过 ollama等）

- ChatOptions(模型参数)：常用的有上下文长度，思考深度（0-1），此外还有各个模型独有的参数。


非常好的教程：
[核心概念-阿里云Spring AI Alibaba官网官网](https://java2ai.com/docs/dev/tutorials/basics/concepts/?spm=5176.29160081.0.0.96aaaa5cCIHmFw#%E5%B0%86%E6%82%A8%E7%9A%84%E6%95%B0%E6%8D%AE%E5%92%8C-api-%E5%BC%95%E5%85%A5-ai-%E6%A8%A1%E5%9E%8B)


嵌入：将文本、图像和视频转换为称为向量（Vectors）的浮点数数组

嵌入模型（Embedding Model）：嵌入过程中采用的模型

文档（Document）：文档是文档内容和元数据的容器。它还包含文档的唯一 ID。文档可以包含文本内容或媒体内容，但不能同时包含两者。它旨在用于从外部源获取数据，作为 spring-ai 的 ETL 管道的一部分。（同时也就是矢量查询的元数据类型）





什么是mcp协议？
在mcp的java-SDK中，我们可以看到它大致分为客户端、连接器、传输层三个层次，那么问题来了，所谓的mcp协议是否是我们常规所认为的http协议？很显然并不是，mcp协议是一种一种编码规范，如果了解过funtion-call(tool),那么我们就明白，mcp协议的目标并不是传输功能，而是提供一组工具给chatclient使用。

*协议并非只有网络协议，对于编码规范，或许我们仍然可以将其视为协议。*

编码定义：Spring AI MCP 为模型上下文协议提供 Java 和 Spring 框架集成
[模型上下文协议（Model Context Protocol）-阿里云Spring AI Alibaba官网官网](https://java2ai.com/docs/1.0.0-M6.1/tutorials/mcp/?spm=5176.29160081.0.0.96aaaa5cTo2GaP)


mcp编码的两大实体： mcp客户端和mcp服务端
1，通信使用mcp上下文协议
2，客户端与服务器一对一链接
