## redis进阶

### 数据备份和恢复
Redis SAVE 命令用于创建当前数据库的备份。

```
redis 127.0.0.1:6379> SAVE
OK
```

该命令将在 redis 安装目录中创建dump.rdb文件。

### 性能测试
Redis 性能测试是通过同时执行多个命令实现的。

用自带的redis-benchmark

```
$ redis-benchmark -n 10000  -q

PING_INLINE: 141043.72 requests per second
PING_BULK: 142857.14 requests per second
SET: 141442.72 requests per second
GET: 145348.83 requests per second
INCR: 137362.64 requests per second
LPUSH: 145348.83 requests per second
LPOP: 146198.83 requests per second
SADD: 146198.83 requests per second
SPOP: 149253.73 requests per second
LPUSH (needed to benchmark LRANGE): 148588.42 requests per second
LRANGE_100 (first 100 elements): 58411.21 requests per second
LRANGE_300 (first 300 elements): 21195.42 requests per second
LRANGE_500 (first 450 elements): 14539.11 requests per second
LRANGE_600 (first 600 elements): 10504.20 requests per second
MSET (10 keys): 93283.58 requests per second
```

结果显示每秒执行的请求数。

### redis分区
分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存key的一个子集。


#### 优势

 - 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
 - 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

#### 劣势
redis的一些特性在分区方面表现的不是很好：

 - 涉及多个key的操作通常是不被支持的。举例来说，**当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作**。
 - **涉及多个key的redis事务不能使用。**
 - 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
 - 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。然而，一种叫做presharding的技术对此是有帮助的。

#### 范围分区
最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。

比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。
这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。

#### 哈希分区
另外一种分区方法是hash分区。这对任何key都适用
