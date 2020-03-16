## openssl 详解
OpenSSL 是一个安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用．

OpenSSL 是一个开源项目，其组成主要包括一下三个组件：

 - openssl：多用途的命令行工具
 - libcrypto：加密算法库
 - libssl：加密模块应用库，实现了ssl及tls

其中openssl组件可以实现：秘钥证书管理、对称加密和非对称加密。

**zwlj:简而言之就是利用openssl命令行工具，我们可以直接用命令行就进行hash 密钥加密等操作**

example：

```
echo -n "http://cgi.gamesafe.oa.com/cgi/internal/wechat/account/manage?access_key_id=E1UV3XU2HY4B&expire_time=1534489999&body_md5=F95AC759C918B78B48A73805EFB20577" | openssl sha1 -hmac "RJUK1RAR7TFC2QBXIMN4BNS5DAABIGRA" -binary | openssl base64
```

上面就是把一个url地址，用sha1的方式来进行hmac(也就是用sha1哈希算法哈希后的数据用 access_key 作为对称密钥加密),产生出的字符串又经过base64编码。