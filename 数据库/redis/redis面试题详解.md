## redis面试题详解

#### MySQL里有2000w数据，Redis中只存20w的数据，如何保证Redis中的数据都是热点数据（redis有哪些数据淘汰策略？？？）

相关知识：redis 内存数据集大小上升到一定大小的时候，就会施行 **数据淘汰策略（回收策略）**。redis 提供 6种数据淘汰策略：

 - volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
 - volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
 - volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
 - allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
 - allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
 - no-enviction（驱逐）：禁止驱逐数据

#### Redis的并发竞争问题如何解决?
Redis为单进程单线程模式，采用队列模式将并发访问变为串行访问。Redis本身没有锁的概念

Redis对于多个客户端连接并不存在竞争，但是在Jedis客户端对Redis进行并发访问时会发生连接超时、数据转换错误、阻塞、客户端关闭连接等问题，这些问题均是由于客户端连接混乱造成。对此有2种解决方法：


1. 客户端角度，为保证每个客户端间正常有序与Redis进行通信，对连接进行池化，同时对客户端读写Redis操作采用内部锁synchronized。(也就是自己解决)  

2. 服务器角度，利用setnx实现锁。

### 参考
[面试中关于Redis的问题看这篇就够了](https://juejin.im/post/5ad6e4066fb9a028d82c4b66)
