## ssl协议
SSL(Secure Sockets Layer 安全套接层),及其继任者传输层安全（Transport Layer Security，TLS）是为网络通信提供**安全**及**数据完整性**的一种安全协议。TLS与SSL在传输层对网络连接进行加密。

zwlj：TLS协议是SSL协议的继任者，都可以保障数据安全和完整。

SSL协议可分为两层： SSL记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。 SSL握手协议（SSL Handshake Protocol）：它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

SSL/TLS的基本过程其实就是分三个步骤：

1. 客户端向服务器端索要并验证公钥。
2. 双方协商生成"对话密钥"。
3. 双方采用"对话密钥"进行加密通信。

其中前两步也叫握手阶段，https笔记里也有详细说到这几个步骤。

![](image/ssl0.png)