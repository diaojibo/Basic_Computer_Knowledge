### Boost协程

 boost coroutine2库（boost coroutine已经废弃，建议使用boost coroutine2）为cpp提供协程支持 



协程有如下特点：

1. 同其他数据类型一样，协程也是第一类（first-class）对象，可以被当参数传递等操作；
2. 运行特点是挂起运行，离开协程，过后再进入，恢复运行；
3. **具有对称和非对称的转移控制机制**
4. 挂起前和恢复后本地变量的值是一致的；
5. **有stackless和stackful两种类型**

**ps 区分有栈协程和无栈协程，以及对称协程与非对称协程很重要**



#### 对称与非对称

最大的区别就是，协程切换的时候，是否知道下一个要调用的是哪个协程。

非对称协程知道它的调用方，使用特殊操作隐式地向其调用方提供控制,一个非对称协程可以看 做是从属于它的调用者的。反之对于对称协程，所有对称协程都是等价的，一个对称协程下次可能会跳到任何一个协程里去。

 这两个概念是等价的，一个完全通用的协程库可以提供对称和不对称两种方式。 



#### 无栈协程

stackless的的无栈协程，只能保存最先调用的协程的堆栈，如果协程里面又开别的协程，那堆栈数据不会被保存。



 boost::coroutines2::coroutine<>被用来实现协程。它有两个嵌套类型：pull_type和push_type。

- pull_type可以从push_type那里接收并返回数据。
- push_type可以把数据传给pull_type。 



### Spawn

Boost中要用到协程肯定就离不开spwan操作了。

boost::asio::spawn()创建一个协程，使C++网络编程大大简化，

**boost::asio::spawn(my_strand, do_echo);**

**一般输入2个参数，参数1是 io_service 或者是 strand**，

strand是协程的线性执行器类型，用以指示协程任务在单线程中线性执行。

io_context则是asio中接管io任务的类。



参数2是协程函数，类型如下：
void coroutine(boost::asio::yield_context yield);

一般协程函数都要带一个参数，这个是boost::asio::yield_context类型，这个yield context就代表着当前执行的协程，每次协程任务被执行时都会被传递进去。再协程内部调用其他协程函数的时候，依旧还是要把这个yield_context带上，这样其实可以让下一个协程知道，是谁发起的你，等你调用完了，要跳回到之前的协程上下文继续运行。



```cpp
#include <boost/asio/io_service.hpp>
#include <boost/asio/spawn.hpp>
#include <boost/asio/write.hpp>
#include <boost/asio/buffer.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <list>
#include <string>
#include <ctime>

using namespace boost::asio;
using namespace boost::asio::ip;

io_service ioservice;
tcp::endpoint tcp_endpoint{tcp::v4(), 2014};
tcp::acceptor tcp_acceptor{ioservice, tcp_endpoint};
std::list<tcp::socket> tcp_sockets;

void do_write(tcp::socket &tcp_socket, yield_context yield)
{
  std::time_t now = std::time(nullptr);
  std::string data = std::ctime(&now);
  async_write(tcp_socket, buffer(data), yield);
  tcp_socket.shutdown(tcp::socket::shutdown_send);
}

void do_accept(yield_context yield)
{
  for (int i = 0; i < 2; ++i)
  {
    tcp_sockets.emplace_back(ioservice);
    tcp_acceptor.async_accept(tcp_sockets.back(), yield);
    spawn(ioservice, [](yield_context yield)
      { do_write(tcp_sockets.back(), yield); });
  }
}

int main()
{
  tcp_acceptor.listen();
  spawn(ioservice, do_accept);
  ioservice.run();
}
```



上面一上来就先绑定acceptor和一个io context(io service)，然后spawn一个协程投到ioservice