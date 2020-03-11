## 多线程服务端编程模型

### Reactor模式
对于C++网络服务程序，经常使用one loop per thread的模型。

即，**每个IO线程里有一个event loop用于处理读写和定时事件，这种就是Reactor**。