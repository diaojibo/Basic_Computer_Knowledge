## libcurl使用
libcurl作为是一个多协议的便于客户端使用的URL传输库，基于C语言，提供C语言的API接口，支持DICT, FILE, FTP, FTPS, Gopher, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMTP, SMTPS, Telnet and TFTP这些协议，同时支持使用SSL证书的安全文件传输：HTTP POST, HTTP PUT, FTP 上传, 基于HTTP形式的上传、代理、Cookies、用户加密码的认证等多种应用场景。


### 1.全局初始化
程序必须初始化一些libcurl的全局函数。这意味着不管你准备使用libcurl多少次，你都应该，且只初始化一次。当你的程序开始的时候，使用`curl_global_init()`

这个函数需要一个参数来告诉它如何来初始化。使用CURL_GLOBAL_ALL ,它将用通常是比较好的默认参数来初始化所有已知的内部子模块。

```
flags:
    CURL_GLOBAL_ALL //初始化所有内部的模块
    CURL_GLOBAL_SSL //初始化支持安全套接字
    CURL_GLOBAL_WIN32 //初始化win32套接字库
    CURL_GLOBAL_NOTHING //不初始化任何额外模块
    CURL_GLOBAL_DEFAULT //与CURL_GLOBAL_ALL相同
```

libcurl有个默认的保护机制，检测如果curl_global_init没有在curl_easy_perform之前被调用，那么libcurl会猜测该使用的初始化模式来执行程序。**请注意，依靠默认的保护机制来这么做一点都不好**。

当程序不再使用libcurl，请调用curl_global_cleanup函数来对应初始化函数所做的工作，它会做逆向的工作来清理curl_global_init所初始化的资源。

**请避免重复的调用curl_global_init和 curl_global_cleanup。他们每个仅被调用一次。**

### 2.使用easy interface
libcurl中被称为easy interface的api函数，所有这些函数都是有相同的前缀：curl_easy 。


#### 初始化handler

要使用easy interface，首先必须创建一个easy handle，easy handle用于执行每次操作。基本上，每个线程都应该有自己的easy handle用于数据通信（如果需要的话）。千万不要在多线程之间共享同一个easy handle。

``` c++
CURL *curl_handler = curl_easy_init();
```

##### 设置参数

创建好句柄以后就设置curl的属性

libcurl针对所有协议，统一使用一个简单的函数**curl_easy_setopt**进行设置，这大大简化了使用，不过需要参考不同协议设置不同的属性。这里以http发送GET请求为例设置如下：

``` c++
//设置请求的url
curl_easy_setopt(curl_handler, CURLOPT_URL, url);

//设置是否返回请求头
curl_easy_setopt(curl_handler, CURLOPT_HEADER, 1L);

//设置屏蔽其他信号
curl_easy_setopt(curl_handler, CURLOPT_NOSIGNAL, 1L);

//设置下载数据回调函数
curl_easy_setopt(curl_handler, CURLOPT_WRITEFUNCTION, write_func);
curl_easy_setopt(curl_handler, CURLOPT_WRITEDATA, write_data);
回调函数原型：
    size_t function( void *ptr, size_t size, size_t nmemb, void *userp);
        函数将在libcurl接收到数据后被调用。
        void *ptr是下载回来的数据.
        void *userp是用户指针, 用户通过这个指针传输自己的数据.
CURLOPT_WRITEDATA：设置回调函数中的void *userp指针的来源。

//设置是否使用下载进度控制函数
curl_easy_setopt(curl_handler, CURLOPT_NOPROGRESS, 0L);
curl_easy_setopt(curl_handler, CURLOPT_PROGRESSFUNCTION, prog_func);
curl_easy_setopt(curl_handler, CURLOPT_PROGRESSDATA, pdata);
下载进度回调函数与下载数据的回调函数原型相同，data也相同。

//设置请求头
struct curl_list *header_list = NULL;
for (int i = 0; i < headers.size(); ++i) {
    header_list = curl_slist_append(header_list, headers[i].c_str());
}
curl_easy_setopt(curl_handler, CURLOPT_HTTPHEADER, header_list);
curl_slist_free_all(header_list);

//其他选项
     CURLOPT_HEADERFUNCTION
     CURLOPT_HEADERDATA
       只取HTTP头部数据, 处理与下载数据回调的处理相同. 
     CURLOPT_TIMEOUT
       超时时间.
     CURLOPT_CONNECTIONTIMEOUT
       连接等待时间.
     CURLOPT_FOLLOWLOCATION
　　   设置支持302重定向
　　 CURLOPT_RANGE
    　 断点续传, 指定传输分片, 格式:"0-200"

```

其中，针对http协议，不同请求有特定的属性需要设置，下面依次列举： 

- HEAD：需要设置CURLOPT_NOBODY为true 
- DELETE：需要设置CURLOPT_CUSTOMREQUEST为”DELETE” 
- PUT：需要设置CURLOPT_UPLOAD为true，同时设置CURLOPT_READFUNCTION、CURLOPT_READDATA、CURLOPT_INFILESIZE_LARGE 
- POST：需要设置CURLOPT_POST为true，设置CURLOPT_POSTFIELDS、CURLOPT_POSTFIELDSIZE


#### 执行

``` c++
CURLcode curl_easy_perform(CURL *handler);
```

该函数执行当次请求创建的句柄，返回值有非常详细的定义[libcurl库返回状态码解释与速查](https://blog.csdn.net/u010487568/article/details/52624477)。

#### 销毁当次请求句柄

``` c++
void curl_easy_cleanup(CURL *handler);
```

#### 全局析构

``` c++
void curl_global_cleanup();
```

#### 其他接口

libcurl除了上述常用接口外，提供了其他接口可以进行更为方便和精确的控制。

``` c++
CURLcode curl_easy_getinfo(CURL *curl, CURLINFO info, ... );
//支持的CURLINFO类型见此：https://curl.haxx.se/libcurl/c/curl_easy_getinfo.html

curl_version()  returns a pointer to the libcurl version string
curl_getdate()  converts a date string to time_t
curl_formadd()  build multipart form-data posts
curl_formfree() free a previously built form POST
curl_slist_append() builds a linked list
curl_slist_free_all()   frees a whole curl_slist as made with curl_slist_append()
curl_easy_escape()  URL encodes a string
curl_easy_unescape()    URL decodes a string

```