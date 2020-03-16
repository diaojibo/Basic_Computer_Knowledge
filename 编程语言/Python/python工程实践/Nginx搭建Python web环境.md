## Nginx搭建python web环境
在讨论Python web的环境之前，有几个概念还是要搞清楚。

在网络的cgi笔记里，我们知道了cgi程序其实就是web服务器开启的一个进程，用来处理http请求。那么我们看到的WSGI，uwsgi，uWSGI这些又是啥玩意呢？

看一个流程图

![](image/wsgi0.jpg)

client发送请求以后，请求给到nginx，nginx是一个web服务器，然后nginx会把这个请求传给uwsgi，注意，这个uwsgi**也是一个web服务器**，uwsgi这个web服务器会把这个请求交给flask app进行处理，处理好以后返回结果。

因此我们知道：

 - ngnix，一种高性能web服务器，用来接收请求、处理请求、返回响应
 - flask，python的web框架，处理python请求，我们的代码就写在这里面
 - uWSGI, 实现了uwsgi协议和WSGI的web服务器。在上面的流程中，协助nginx和flask通信。


zwlj：这里注意一下WSGI，uWSGI,uwsgi：

 - **WSGI（Web Server Gateway Interface）**，是为Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口协议，只要web服务器和 web应用都遵守WSGI协议，那么两者之间就可以任意组合。
 - **uwsgi**，是基于二进制的线路协议，与WSGI协议作用相同，但属于uWSGI服务器自有协议。也就是说是一种服务器协议。
 - **uWSGI是Web服务器**，**它实现了WSGI协议、uwsgi、http等协议**

简而言之，uWSGI是一个服务器，uwsgi和WSGI都是协议。

### 安装nginx
很多方法安装nginx，在ubuntu下应该

```
sudo apt-get install nginx
```

就可以了，然后找到nginx的可执行文件，执行

```
sudo nginx -t
```


可以看到配置文件的位置。

这部分的更进一步操作可以单独看ngnix笔记。

### 使用Flask

Flask的基本使用非常简单，也就不多做介绍了。一个基本的Flask项目，会把一个app.py文件暴露在外，我们运行`python app.py`文件的时候，就会自动启用一个python测试服务器，开启一些端口进行测试。

使用Flask默认的服务器是不健壮的。我们一般通过引入uWSGI服务器来来部署Flask应用，并且把response转接给Nginx。

### 使用uwsgi
uwsig其实也相当于一个python库，直接pip安装就可以了。

```
pip install uwsig
```

这里也强烈建议使用pipenv虚拟环境做整合。

启动uwsgi，主要依靠配置文件，这里也主要记录一下配置文件的注意项。

uwsgi配置文件，一般是ini格式的。以下是一个示例 wsgi.ini：

```
[uwsgi]
module = app:app

buffer-size=32768

master = true
processes = 3

http-socket = 127.0.0.1:30001 ;本地http测试端口
socket = 127.0.0.1:30000 ; nginx转发地址 

chmod-socket = 777
vacuum = true

# log文件地址，设置后使得uwsgi进程进入后台
daemonize = ./log/uwsgi.log
chdir = ./app

pidfile=uwsgi.pid

die-on-term = true

; 代码修改后自动重启
python-autoreload=1
```

大部分都可以比较直观的理解，但是也要注意几个：

 - app:app是用来取Flask对象的，第一个app指的是app.py文件，也就是这个module模块。第二个app指的是这个文件里的app对象，如果叫其他名字也应该改为其他。
 - chdir，指的是app.py所在的目录，这里要写对才能正确找到module。
 - daemonize，log文件的地址，配置了这个以后就能把日志信息输出到这里。注意也可以用logto，用logto的话，uwsgi则不会变为后台进程，用daemonize的话会自动变为后台进程。

## 参考
[Linux搭建Python web环境（nginx + flask + uwsgi)](https://www.jianshu.com/p/85692a94e99b)