## Python协程
Python在3.5版本中引入了关于协程的语法糖async和await

通常在Python中我们进行并发编程一般都是使用多线程或者多进程来实现的，对于CPU计算密集型任务由于GIL的存在通常使用多进程来实现，而对于IO密集型任务可以通过线程调度来让线程在执行IO任务时让出GIL，从而实现表面上的并发。

其实对于IO密集型任务我们还有一种选择就是协程。协程，又称微线程，英文名Coroutine，是运行在单线程中的“并发”，协程相比多线程的一大优势就是省去了多线程之间的切换开销，获得了更高的运行效率。Python中的异步IO模块asyncio就是基本的协程模块。

Python中的协程经历了很长的一段发展历程。最初的生成器yield和send()语法，然后在Python3.4中加入了asyncio模块，引入@asyncio.coroutine装饰器和yield from语法，在Python3.5上又提供了async/await语法，目前正式发布的Python3.6中asynico也由临时版改为了稳定版。

### 协程
协程笔记里已经有讲概念了，是单个线程内的任务调度技巧，这里讲一下python的协程。

Python中的协程大概经历了如下三个阶段： 
1. 最初的生成器变形yield/send 
2. 引入@asyncio.coroutine和yield from 
3. 在最近的Python3.5版本中引入async/await关键字

#### 生成器变形yield/send

普通函数中如果出现了yield关键字，那么该函数就不再是普通函数，而是一个生成器。

``` python
def mygen(alist):
    while len(alist) > 0:
        c = randint(0, len(alist)-1)
        yield alist.pop(c)
a = ["aa","bb","cc"]
c=mygen(a)
print(c)

输出：<generator object mygen at 0x02E5BF00>
```

像上面代码中的**c就是一个生成器**。**生成器就是一种迭代器**，可以使用for进行迭代。生成器函数最大的特点是可以接受外部传入的一个变量，并根据变量内容计算结果后返回。 

yield和next其实我们在迭代器和生成器的笔记里已经介绍过了，可以利用next不停的生成结果。生成器也是能被for循环的，所以我们能用for循环来不停对生成器进行计算。

最初的yield只能返回并暂停函数，并不能实现协程的功能。后来，Python为它定义了新的功能——接收外部发来的值，**这样一个生成器就变成了协程**。

每个生成器都可以执行send()方法，为生成器内部的yield语句发送数据。此时yield语句不再只是yield xxxx的形式，还可以是var = yield xxxx的赋值形式。它同时具备两个功能，一是暂停并返回函数，二是接收外部send()方法发送过来的值，重新激活函数，并将这个值赋值给var变量！

``` python
def simple_coroutine():
    print('-> 启动协程')
    y = 10
    x = yield y
    print('-> 协程接收到了x的值:', x)

my_coro = simple_coroutine()
ret = next(my_coro)
print(ret)
my_coro.send(10)
```

如上图，我们不仅仅可以next这个生成器，我们甚至可以与这个协程通信，我们能接受值还能send值进去进行流程控制。

#### @asyncio.coroutine与yield from
@asyncio.coroutine：asyncio模块中的装饰器，用于将一个生成器声明为协程。
yield from 其实就是等待另外一个协程的返回。

yield from在asyncio模块中得以发扬光大。之前都是我们手工切换协程，现在当声明函数为协程后，我们通过事件循环来调度协程。

``` python
import asyncio,random
@asyncio.coroutine
def smart_fib(n):
    index = 0
    a = 0
    b = 1
    while index < n:
        sleep_secs = random.uniform(0, 0.2)
        yield from asyncio.sleep(sleep_secs) #通常yield from后都是接的耗时操作
        print('Smart one think {} secs to get {}'.format(sleep_secs, b))
        a, b = b, a + b
        index += 1

@asyncio.coroutine
def stupid_fib(n):
    index = 0
    a = 0
    b = 1
    while index < n:
        sleep_secs = random.uniform(0, 0.4)
        yield from asyncio.sleep(sleep_secs) #通常yield from后都是接的耗时操作
        print('Stupid one think {} secs to get {}'.format(sleep_secs, b))
        a, b = b, a + b
        index += 1

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    tasks = [
        smart_fib(10),
        stupid_fib(10),
    ]
    loop.run_until_complete(asyncio.wait(tasks))
    print('All fib finished.')
    loop.close()
```

这里要千万注意这种协程的运作方法了，首先在main函数里，是创建了一个事件循环，这个事件循环就是协程运作的基础了。创建这个loop之后，就会去循环执行生成器任务，遇到yield操作的时候，不会阻塞，而是重新开始一个循环去执行任务。

所以线程不会等待asyncio.sleep()，而是直接中断并执行下一个消息循环。当asyncio.sleep()返回时，线程就可以从yield from拿到返回值（此处是None），然后接着执行下一行语句。 
asyncio是一个基于事件循环的实现异步I/O的模块。通过yield from，我们可以将协程asyncio.sleep的控制权交给事件循环，然后挂起当前协程；之后，由事件循环决定何时唤醒asyncio.sleep,接着向后执行代码。 
协程之间的调度都是由事件循环决定。 

### async和await
搞清楚前面的概念以后，再来看async await就顺眼多了。可以将他们理解成asyncio.coroutine/yield from的完美替身。当然，从Python设计的角度来说，async/await让协程表面上独立于生成器而存在，将细节都隐藏于asyncio模块之下，语法更清晰明了

加入新的关键字 async ，可以将任何一个普通函数变成协程(正如前面介绍，一个高级版加了send的生成器)：

``` python
import time,asyncio,random
async def mygen(alist):
    while len(alist) > 0:
        c = randint(0, len(alist)-1)
        print(alist.pop(c))
a = ["aa","bb","cc"]
c=mygen(a)
print(c)

输出：
<coroutine object mygen at 0x02C6BED0>
```
在上面程序中，我们在前面加上async，该函数就变成一个协程了。**但是async对生成器是无效的。async无法将一个生成器转换成协程。** 所以我们不能再用yield了，我们应该这样写：

``` python
import time,asyncio,random
async def mygen(alist):
    while len(alist) > 0:
        c = random.randint(0, len(alist)-1)
        print(alist.pop(c))
        await asyncio.sleep(1) 
strlist = ["ss","dd","gg"]
intlist=[1,2,5,6]
c1=mygen(strlist)
c2=mygen(intlist)
print(c1)
```

要运行协程，要用事件循环 
在上面的代码下面加上：

```
if __name__ == '__main__':
        loop = asyncio.get_event_loop()
        tasks = [
        c1,
        c2
        ]
        loop.run_until_complete(asyncio.wait(tasks))
        print('All fib finished.')
        loop.close()
```


**其实使用的时候，我们去想象js那种事件循环就行了。**