### with as
这个语法是用来代替传统的try...finally语法的。 

基本思想是with所求值的对象必须有一个__enter__()方法，一个__exit__()方法。

紧跟with后面的语句被求值后，返回对象的__enter__()方法被调用，这个方法的返回值将被赋值给as后面的变量。当with后面的代码块全部被执行完之后，将调用前面返回对象的__exit__()方法。

好比说open,我们经常忘记去清理环境,那就用with就好了：

``` python
file = open("/tmp/foo.txt")
try:
    data = file.read()
finally:
    file.close()
```

#### 新语法async with
异步上下文管理器使用一种新的语法:

异步上下文管理器指的是在enter和exit方法处能够暂停执行的上下文管理器。意思就差不多是，把对象的enter和exit也做成是一种异步的操作，放到时间循环里，操作没做完的时候就去做别的事。

为了实现这样的功能，需要加入两个新的方法：`__aenter__` 和`__aexit__`。这两个方法都要返回一个 awaitable类型的值。

异步上下文管理器的一种使用方法是:

``` python
class AsyncContextManager:
    async def __aenter__(self):
        await log('entering context')

    async def __aexit__(self, exc_type, exc, tb):
        await log('exiting context')
```

