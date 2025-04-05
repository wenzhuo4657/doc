
[git仓库地址I](https://github.com/modelcontextprotocol/java-sdk?tab=readme-ov-file)
[文档地址](https://modelcontextprotocol.io/sdk/java/mcp-overview)

## 架构
![[Pasted image 20250402144114.png]]
客户端/服务器层（McpClient/McpServer）：都使用 McpSession 进行同步/异步操作，其中 McpClient 处理客户端协议操作，McpServer 管理服务器端协议操作。
会话层（McpSession）：使用 DefaultMcpSession 实现管理通信模式和状态。
传输层（McpTransport）：通过以下方式处理 JSON-RPC 消息序列化/反序列化：
核心模块中的 StdioTransport (stdin/stdout)
专用传输模块中的 HTTP SSE 传输（Java HttpClient、Spring WebFlux、Spring WebMVC）





