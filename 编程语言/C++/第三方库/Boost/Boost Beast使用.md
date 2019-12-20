## Boost Beast使用



### HTTP Client

https://www.boost.org/doc/libs/1_70_0/libs/beast/doc/html/beast/quick_start/http_client.html

同步的流程就是，

 - asio的resolver解析域名和ip
 - 创建beast的tcp stream
 - beast的stream connect 解析出的socket
 - 创建beast http request
 - 设置request头部等相关信息
 - 调用beast http write 写stream
 - 读HTTP stream