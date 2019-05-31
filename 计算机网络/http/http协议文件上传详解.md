## http文件上传
我们经常会要用http协议向服务器上传数据，那么这个操作是怎么样一个过程呢？

上传文件最关键的一个字段就是：**multipart/form-data**，当然客户端要往服务端上传一个文件的时候，它就会在请求头部加上一个header：`Content-Type: multipart/form-data`

这里我们还是要先说一下MIME

### MIME
MIME（Multipurpose Internet Mail Extensions），它并不是HTTP协议的一部分，就像我们每个人都是独一无二的，有自己的属性，互联网上每个资源也有属性，比如有些资源是图片，有些是视频，有些是HTML页面，MIME规定了每种资源的类型，这个类型不是随便定义的，由IANA负责登记和维护。

比如你看到一个URL地址，`http://localhost/image.png`，我们其实并不是通过.png后缀判断资源类型的，而是通过MIME来获知该资源类型的，这个图片的MIME可能就是image/png

MIME类型结构如下：

```
type/subtype
```

type相当于某些类型的集合，而subtype相当于子类型。以image/png为例，image表示图片类型集合，png表示某种类型图片。

让我们看几个比较重要的MIME类型：

```
text/plain
text/html
application/octet-stream
multipart/form-data
```

### Content Type
现在我们定义了每种资源的MIME类型，那么客户端如何知晓每种资源的MIME类型呢？这时候就要使用Content-Type HTTP Header 头了，比如我们请求一个资源，Web服务器在发送资源的时候，发送了“content-type：image/png” Header 头，这样客户端就知道该资源是一个png图片了。

如果客户端发送了一个 “Content-Type: multipart/form-data;”，代表客户端要上传一个附件。

### multipart/form-data
multipart/form-data 这个MIME类型**并不是标准的MIME类型**，而是因为Web的需要扩展而来的

那么multipart/form-data表示什么呢？multipart互联网上的混合资源，就是资源由多种元素组成，form-data表示可以使用HTML Forms 和 POST 方法上传文件，具体的定义可以参考RFC 7578。

从HTTP协议的角度，最后看下文件上传的HTTP消息体，使用Postman也容易看出，如下：

```
POST /api.php HTTP/1.1
Host: localhst
Cache-Control: no-cache
Content-Type: multipart/form-data; boundary=----FormBoundary

------FormBoundary
Content-Disposition: form-data; name="file"; filename="file.png"
Content-Type: image/png

<图片二进制内容>
------FormBoundary
Content-Disposition: form-data; name="param1"

value1
------FormBoundary
Content-Disposition: form-data; name="param2"

value2
------FormBoundary--
```

Content-Type: multipart/form-data; boundary=——FormBoundary 表示要上传附件，其中boundary表示分隔符，如果要上传多个表单项，就要使用boundary分割，每个表单项由———FormBoundary开始，以———FormBoundary结尾。每一个表单项又由Content-Type和Content-Disposition组成。


```
------FormBoundary
Content-Disposition: form-data; name="file"; filename="file.png"
Content-Type: image/png

<图片二进制内容>
------FormBoundary
```

表示普通的一个表单元素(上传的都叫表单元素)，最重要的是理解 Content-Disposition HTTP 消息头，其中第一个参数总是固定不变的form-data，name表示表单元素属性名，回车换行符后面的内容就是元素的值。