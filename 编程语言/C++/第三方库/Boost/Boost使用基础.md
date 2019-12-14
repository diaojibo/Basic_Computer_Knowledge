## Boost使用基础



### Boost编译使用

首先我们需要获取Boost的源码，我们去官网。

找到documentation的下载地址，下载最新版本的boost源码

目前我这个时间点：

![1576310036597](image/1576310036597.png)

根据文档提示在unix机器上解压：

```\
tar --bzip2 -xf /path/to/boost_1_72_0.tar.bz2
```

这个boost的源码文件，竟然有100多兆，嘤嘤嘤

![1576311516451](image/1576311516451.png)

解压后，上面是boost的结构。



**重点来了，boost的大部分库都是header-only的！！所以我们只要把这个库解压出来，就能很方便的使用。只需要在添加依赖的时候正确的引入头文件就好，如`#include <boost/whatever.hpp>`**

