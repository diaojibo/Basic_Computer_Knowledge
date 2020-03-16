## nginx
Nginx("engine x")是一款是由俄罗斯的程序设计师Igor Sysoev所开发高性能的 Web和 反向代理 服务器，也是一个 IMAP/POP3/SMTP 代理服务器。
在高连接并发的情况下，Nginx是Apache服务器不错的替代品。

## nginx安装
在ubuntu下，其实比较好装。

```
sudo apt-get install nginx
```

nginx的可执行文件会自动安装到用户目录下。直接`nginx`就能运行服务器。

nginx执行时会使用默认配置，查看nginx使用配置的位置，用如下命令：

```
sudo nginx -t
```

注意一点，根据安装nginx方式的不同(yum安装或者源码安装)，配置文件结构将会有比较大的不同。ubuntu的apt-get安装，则配置文件一般在`/etc/nginx`下。

同时这个地方还有两个子目录，用来放子配置文件,这些配置文件会被主配置文件include

 - sites-available – 包含为多个网站准备的多个配置文件。
 - sites-enabled – 包含一个指向 sites-available 目录中配置文件的软链接。


#### 源码安装nginx
有时候我们用不了包管理软件的时候，就要用源码安装nginx了。

首先我们要安装PCRE

```
cd /usr/local/src/
wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
tar zxvf pcre-8.35.tar.gz
cd pcre-8.35
./configure
make && make install
```

如此一来我们就安装了pcre了。


安装nginx：

```
cd /usr/local/src/
wget http://nginx.org/download/nginx-1.6.2.tar.gz

tar zxvf nginx-1.6.2.tar.gz
cd nginx-1.6.2
./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35

make
make install
```

注意上面在编译的时候，指定了prefix，也就是nginx的安装目录，在/usr/local/webserver/nginx这里。

用下面命令看版本。`/usr/local/webserver/nginx/sbin/nginx -v`


## nginx配置详解

Nginx 的顶层配置文件是 nginx.conf。Nginx 接受多层级的配置文件，这也使得用户可以针对自己的应用进行弹性的配置。

这些当然可以通过官方文档了解：[地址](http://nginx.org/en/docs/ngx_core_module.html)

在 Nginx 中，由配置块（block）来组织各个配置参数。以下为在本文中我们将提到的配置块：

 - Main – 定义于 nginx.conf（所有不属于配置块的参数均属 Main 块）
 - Events – 定义于 nginx.conf
 - Http – 定义于 nginx.conf
 - Server – 定义于 application_name.conf

接下来分为每个部分细说一下：

### Main部分
main配置就是那些全局配置，不被Events块和Http块包起来那些外围设置，就是Main部分。好比如：

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
```

 - 第一个参数（user）将定义 Nginx 服务器的拥有者以及运行用户。当 Nginx 通过 Docker 容器运行时，使用默认值就够了。
 - 第二个参数（worker_processes）定义了 worker processes（工作进程）的数量。此参数推荐的默认值为当前服务器使用内核的数量。对于基础的虚拟私有服务器（VPS）来说，默认值 1 就是个不错的选择。当你拓展 VPS 性能时可以增加这个数字。
 - 第三个参数（error_log）定义了错误日志在文件系统中存放的位置，并能额外定义一个参数来规定需要记录日志的最小错误等级。这个参数使用默认值即可。
 - 第四个参数（pid）定义了用于存储 Nginx 主进程 pid 的文件位置。这个参数使用默认值即可。


### events块

events 配置块定义了一些会影响连接处理的参数。它也是 Nginx.conf 文件中第一个配置块：

```
events {
    worker_connections  1024;
}
```

在这个配置块中仅有一个单独的参数（worker_connections），定义了工作进程可以打开的最大并发连接数。默认值定义了总共可用 1024 个连接，无需更改（但你需要计算用户请求站点及请求 WSGI 服务器的连接数）。


### http块
http 配置块定义了一些关于 Nginx 如何处理 HTTP Web 流量的参数。它是 nginx.conf 文件中第二个配置块：


```
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

 - 第一个参数（include）指定了需要引入的配置文件，mime.types 文件，这个文件定义了各种 Nginx 支持的文件类型。此参数应该保持默认值。
 - 第二个参数（default_type）指定了默认给用户返回的文件类型。
 - 第三个参数（log_format）指定了日志的格式，应当保持默认值。
 - 第四个参数（access_log）指定了 Nginx 日志的访问位置，应当保持默认值。
 - 第五个参数（send_file）以及第六个参数（tcp_nopush）稍微有点复杂。详细设置可查阅资料(待补充)
 - 第七个参数（keepalive_timeout）定义了与客户端保持连接的超时时长，应当保持默认值。
 - 第八个参数（gzip）定义了 gzip 压缩算法的使用方法，以减少传输数据量。虽然数据量减少了，但也因此增加平台在压缩过程中的性能消耗，好处两两抵消，因此保持它的默认值（off）。
 - 第九个，也是最后一个参数（include）定义了位于 /etc/nginx/conf.d/ 下后缀名为 .conf 的其它配置文件。现在我们将使用这些配置文件定义静态内容服务器以及 WSGI 服务器的反向代理。

### application的配置

接下来简单讲讲应用的配置，也就是Server块，这个一般可以通过include来把配置引入。

```
# Define the parameters for a specific virtual host/server
server {
   # Define the directory where the contents being requested are stored
   # root /usr/src/app/project/;

   # Define the default page that will be served If no page was requested
   # (ie. if www.kennedyfamilyrecipes.com is requested)
   # index index.html;

   # Define the server name, IP address, and/or port of the server
   listen 80;
   # server_name xxx.yyy.zzz.aaa

   # Define the specified charset to the “Content-Type” response header field
   charset utf-8;

   # Configure NGINX to deliver static content from the specified folder
   location /static {
       alias /usr/src/app/project/static;
   }

   # Configure NGINX to reverse proxy HTTP requests to the upstream server (Gunicorn (WSGI server))
   location / {
       # Define the location of the proxy server to send the request to
       proxy_pass http://web:8000;

       # Redefine the header fields that NGINX sends to the upstream server
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

       # Define the maximum file size on file uploads
       client_max_body_size 5M;
   }
}

```

很多项都一目了然，这里重点讲几个要注意的。

 - location配置块(/static)定义了 Nginx 需要递送位于以下位置的静态内容,比如请求静态资源 www.xxx.com/static，就会对应location配置的static目录去找。
 - location配置块(/)定义的是反向代理的内容。
    1. proxy_pass定义了转发服务器的位置，好比上面配置转发到了本机8000端口。
    2.  proxy_pass_header用来设置转发数据的一些头部。可以看出也用了些内置变量来表示这些参数。
    3.  client_max_body_size，定义了文件上传允许的最大大小，对于需要上传文件的 Web 应用来说非常重要。由于图像大小一般在 2 MB 内，因此在这儿设置 5 MB 基本上可以满足任何图像。





## 参考

[如何为 Flask Web 应用配置 Nginx](https://juejin.im/post/5a795febf265da4e94499949)