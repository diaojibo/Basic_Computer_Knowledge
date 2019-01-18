## python日志模块实践
日志很重要，但是打印输出却不是个好办法。

直接print输出会影响标准输出中查看其它输出数据，所以应该用python的标准日志模块来做处理。


使用日志模块的优点如下：

 - 你可以控制消息的**级别**，过滤掉那些并不重要的消息。
 - 你可决定输出到什么地方，以及怎么输出。


### 日志模块

``` python
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
 
logger.info('Start reading database')
# read database here
 
records = {'john': 55, 'tom': 66}
logger.debug('Records: %s', records)
logger.info('Updating records ...')
# update records here
 
logger.info('Finish updating records')
```

甚至可以用handler来写到文件，或者其它远程服务当中：

``` python
import logging
 
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
 
# create a file handler
 
handler = logging.FileHandler('hello.log')
handler.setLevel(logging.INFO)
 
# create a logging format
 
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
 
# add the handlers to the logger
 
logger.addHandler(handler)
 
logger.info('Hello baby')
```

#### 请使用__name__作为logger名称
虽然不是非得将 logger 的名称设置为 __name__ ，但是这样做会给我们带来诸多益处。在 python 中，变量 __name__ 的名称就是当前模块的名称。比如，在模块 “foo.bar.my_module” 中调用 logger.getLogger(__name__) 等价于调用logger.getLogger(“foo.bar.my_module”) 。当你需要配置 logger 时，你可以配置到 “foo” 中，这样包 foo 中的所有模块都会使用相同的配置。当你在读日志文件的时候，你就能够明白消息到底来自于哪一个模块。

#### 捕捉异常并用traceback记录它
出问题的时候记录下来是个好习惯，但是如果没有 traceback ，那么它一点儿用也没有。你应该捕获异常并用 traceback 把它们记录下来。比如下面这个例子：

``` python
try:
    open('/path/to/does/not/exist', 'rb')
except (SystemExit, KeyboardInterrupt):
    raise
except Exception, e:
    logger.error('Failed to open file', exc_info=True)
```

## 参考
[每个 Python 程序员都要知道的日志实践](http://python.jobbole.com/81666/)