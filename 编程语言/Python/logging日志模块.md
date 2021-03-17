## logging日志模块

logging模块是Python的一个标准库模块，由标准库模块提供日志记录API的关键好处是所有Python模块都可以使用这个日志记录功能。所以，你的应用日志可以将你自己的日志信息与来自第三方模块的信息整合起来。

logging模块提供了两种记录日志的方式：

- 第一种方式是**使用logging提供的模块级别的函数**
- 第二种方式是**使用Logging日志系统的四大组件**

简单介绍一下第一种方式，就是最简单的方式

```python
import logging

logging.debug("This is a debug log.")
logging.info("This is a info log.")
logging.warning("This is a warning log.")
logging.error("This is a error log.")
logging.critical("This is a critical log.")
```

 logging模块提供的模块级别的那些函数实际上也是通过这几个组件的相关实现类来记录日志的，只是在创建这些类的实例时设置了一些默认值。

### 四大组件

| 组件名称 | 对应类名  | 功能描述                                                     |
| -------- | --------- | ------------------------------------------------------------ |
| 日志器   | Logger    | 提供了应用程序可一直使用的接口                               |
| 处理器   | Handler   | 将logger创建的日志记录发送到合适的目的输出                   |
| 过滤器   | Filter    | 提供了更细粒度的控制工具来决定输出哪条日志记录，丢弃哪条日志记录 |
| 格式器   | Formatter | 决定日志记录的最终输出格式                                   |

logging模块就是通过这些组件来完成日志处理的，上面所使用的logging模块级别的函数也是通过这些组件对应的类来实现的。

- 日志器（logger）需要通过处理器（handler）将日志信息输出到目标位置，如：文件、sys.stdout、网络等；
- 不同的处理器（handler）可以将日志输出到不同的位置；
- 日志器（logger）可以设置多个处理器（handler）将同一条日志记录输出到不同的位置；
- 每个处理器（handler）都可以设置自己的过滤器（filter）实现日志过滤，从而只保留感兴趣的日志；
- 每个处理器（handler）都可以设置自己的格式器（formatter）实现同一条日志以不同的格式输出到不同的地方。

因此filter用于过滤日志，formatter用于将日志格式化

### 并发日志写入实例

假定我们有多个进程脚本要往一个日志文件写日志，该怎么操作呢

首先我们引入了一个第三方库 concurrent_log_handler

我们需要预先安装一下，这个库可以让多进程并发写日志：

```python
import logging
from concurrent_log_handler import ConcurrentRotatingFileHandler

# 设置python脚本的切割日志
logger = logging.getLogger()
rotate_handler = ConcurrentRotatingFileHandler('./log/test.log', 'a', 100 * 1024 * 1024, 10)
rotate_handler.setFormatter(logging.Formatter("%(asctime)s - %(levelname)s - %(filename)s[:%(lineno)d] - %(message)s"))
logger.addHandler(rotate_handler)
logger.setLevel(logging.DEBUG)
```

利用到两个组件，一个是log的formatter，这个formatter可以直接用logging.Formatter初始化