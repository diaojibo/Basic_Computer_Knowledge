## Go Http

对于go的http操作，其实使用github上的req包，已经可以满足日常使用了。

但是这里还是再记录一下transport包的使用

net/http 包提供了最简洁的 HTTP 客户端实现，无需借助第三方网络通信库（比如 libcurl）就可以直接使用最常见的 GET 和 POST 方式发起 HTTP 请求。**req包，也是基于net底层进行封装的**

但有时候我们必须定制化一些http的客户端参数，比如如果我们http客户端不做限制，然后开了很多协程进行请求，那很容易会把socket耗尽。哪怕我们对协程数做了限制，不对http请求连接池做限制的话也会有问题。

**ps: 之前就遇到场景，控制了一定数量的协程跑http请求，结果socket还是耗尽了。原因是因为默认的http请求缓存连接数很小，只要超出了这个值，每次http请求的时候都会需要新建一个连接**

ps2：k8S的CLB，针对这个问题做了优化，每次http短连接，CLB会负责断开这个连接，这样客户端就不会出现大量timewait的情况



针对http调参，我们可以用transport来进行http控制

在Go语言标准库中，http.Client 类型包含了 3 个公开数据成员：

```go
Transport RoundTripper
CheckRedirect func(req *Request, via []*Request) error
Jar CookieJar
```

Transport 类型必须实现 http.RoundTripper 接口。Transport 指定了执行一个 HTTP 请求的运行机制，倘若不指定具体的 Transport，默认会使用 http.DefaultTransport，这意味着 http.Transport 也是可以自定义的。net/http 包中的 http.Transport 类型实现了 http.RoundTripper 接口。

http.RoundTripper 接口很简单，只定义了一个名为 RoundTrip 的方法。任何实现了 RoundTrip() 方法的类型即可实现 http.RoundTripper 接口。 http.Transport 类型正是实现了 RoundTrip() 方法继而实现了该接口。

### 设置http transport

```go
r := req.New()
		transportCfg := &http.Transport{
			DialContext: (&net.Dialer{
				Timeout:   10 * time.Second,
				KeepAlive: 30 * time.Second,
				DualStack: true,
			}).DialContext,
			MaxConnsPerHost:     5000,
			MaxIdleConnsPerHost: 5000,
			TLSClientConfig:     &tls.Config{InsecureSkipVerify: true},
		}
		r.Client().Transport = transportCfg
```

其中，MaxConnsPerHost就是用来控制针对访问的每一个host，连接池数缓存多大。假如基本都是对一个host发请求，那可以建议这一个选项调成跟最大连接数一致的数目