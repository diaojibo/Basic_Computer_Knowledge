## Asio使用概况



### resolver

```cpp
boost::asio::io_service io_service ;
boost::asio::ip::tcp::resolver resolver(io_service);
```

tcp::resolver一般和tcp::resolver::query结合用，通过query这个词顾名思义就知道它是用来查询socket的相应信息，一般而言我们关心socket的address和port，

通过tcp::resolver很容易实现设置和查询，它通过query把字符串格式的ip如192.168.0.200或主机名http://localhost，端口“8080”等转化成socket内部表示格式，这样我们应用的时候可以直接使用字符串的形式，**而且不用再担心socket的字节顺序转化问题**。



比如以下用法

```
// Look up the domain name
auto const results = resolver.resolve(host, port);
// Make the connection on the IP address we get from a lookup
stream.connect(results);
```

