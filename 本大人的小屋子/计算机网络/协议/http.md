参考文章：https://segmentfault.com/a/1190000042255540


# http和https区别

HTTPS（HyperText Transfer Protocol Secure）是HTTP的安全版本，它确实使用了SSL（Secure Sockets Layer）或其继任者TLS（Transport Layer Security）来加密数据。不过，SSL/TLS并不是直接在TCP之上的一个协议层，而是位于应用层（HTTP）和传输层（TCP）之间的一个协议层。

具体来说，HTTPS的工作流程如下：

TCP连接建立：客户端和服务器首先通过TCP协议建立一个连接。

SSL/TLS握手：在TCP连接建立之后，客户端和服务器会进行SSL/TLS握手，协商加密算法、交换密钥等，以建立一个安全的通信通道。

加密通信：握手完成后，所有的HTTP数据都会通过这个加密的通道进行传输，确保数据的机密性和完整性。

因此，HTTPS可以理解为HTTP over SSL/TLS。SSL/TLS协议层在HTTP和TCP之间，负责加密和解密数据。


**TLS和SSL一样吗？**
SSL更新到3.0时，IETF对SSL3.0进行了标准化，并添加了少数机制(但是几乎和SSL3.0无差异)，标准化后的IETF更名为TLS1.0,可以说TLS就是SSL的新版本3.1。
TLS是SSL的标准化后的产物  有1.0 1.1 1.2三个版本  默认使用1.0。


所以事实上我们现在使用的是TLS，但由于历史原因，更多将其称为SSL。










