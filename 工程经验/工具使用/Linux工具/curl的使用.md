## curl
curl命令是一个利用URL规则在命令行下工作的**文件传输工具**。它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。作为一款强力工具，**curl支持包括HTTP、HTTPS、ftp等众多协议**，还支持POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。做网页处理流程和数据检索自动化，curl可以祝一臂之力。

### http

直接get

```
curl www.xxx.com
```

用post：

```
curl -d '{"data":123}' www.xxxx.com
```

上传文件：

```
curl --form "file=@/home/rockctli/dev/win_dev/temp_dev/batch_test" 127.0.0.1:8333/batch_punish
```

#### expect 100-continue
The Expect: 100-continue header is specified in HTTP 1.1 and allows the server to acknowledge or reject a POST/PUT request immediately after the headers are send but before the client starts sending potentially large amounts of actual data (body of the request). 

也就是说要上传大文件的时候，curl可能会自动加上Expect 100这个头部，这个头部允许服务器拒绝掉这个请求。所以我们如果要避免这个情况，那就要设置头部来避免：

The expect logic can easily be disabled via setting the Expect: header to the empty string.

```
$ curl -H 'Expect:' -H 'Transfer-Encoding: ...' -H 'Content-Type: ...' \
    --data-binary '@mediumfile' http://example.org/archive -v
```