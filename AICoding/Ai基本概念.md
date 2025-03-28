
- VectorStore(矢量数据库)：检索数据，用于给ai提供数据。 例如，PgVector，

对于ai来讲，这个数据库的作用仅仅是用于提供数据例如springai所提供的简单实现。[SimpleVectorStore](https://github.com/spring-projects/spring-ai/blob/main/spring-ai-core/src/main/java/org/springframework/ai/vectorstore/SimpleVectorStore.java)




- ai服务提供源：各大厂商的token,或者自建的大模型（可通过 ollama等）

- ChatOptions(模型参数)：常用的有上下文长度，思考深度（0-1），此外还有各个模型独有的参数。


非常好的教程：
[核心概念-阿里云Spring AI Alibaba官网官网](https://java2ai.com/docs/dev/tutorials/basics/concepts/?spm=5176.29160081.0.0.96aaaa5cCIHmFw#%E5%B0%86%E6%82%A8%E7%9A%84%E6%95%B0%E6%8D%AE%E5%92%8C-api-%E5%BC%95%E5%85%A5-ai-%E6%A8%A1%E5%9E%8B)



