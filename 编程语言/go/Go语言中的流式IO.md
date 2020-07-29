## 流式IO

在 Go 中，输入输出操作是通过能读能写的字节流数据模型来实现的。为此，io 包提供了 io.Reader 和 io.Writer 接口来进行输入输出操作

![img](image/v2-da53523e772285dd79ee560b64023025_720w.png)

Go 附带了许多 API，这些 API 支持来自内存结构，文件，网络连接等资源的流式 IO。

### io.Reader

由 io.Reader 接口表示的读取器将数据从某些源读取到缓冲区，可以像用水管输送水流一样来传送它，如下所示

对于要用作读取器的类型，它必须从接口 io.Reader 实现 Read(p [] byte)方法，如下所示：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Read() 方法的实现应返回读取的字节数或发生的错误。如果数据源已输出全部内容，则 Read 应返回 io.EOF

直接从读取器流式传输数据很容易。Read 方法被设计为在循环内调用，每次迭代时，它从源读取一大块数据并将其放入缓冲区 p 中。直到 Read 方法返回io.EOF 错误

#### 一个自定义的io reader

以下是 io.Reader 的[简单实现](https://link.zhihu.com/?target=https%3A//github.com/vladimirvivien/learning-go/blob/master/tutorial/io/alpha_reader.go)，它从流中过滤掉非字母字符。

```go
// alphaReader is a simple implementation of an io.Reader
// that streams only alpha chars from its string source.
type alphaReader struct {
    src string
    cur int
}

func newAlphaReader(src string) *alphaReader {
    return &alphaReader{src: src}
}

func alpha(r byte) byte {
    if (r >= 'A' && r <= 'Z') || (r >= 'a' && r <= 'z') {
        return r
    }
    return 0
}

func (a *alphaReader) Read(p []byte) (int, error) {
    if a.cur >= len(a.src) {
        return 0, io.EOF
    }

    x := len(a.src) - a.cur
    n, bound := 0, 0
    if x >= len(p) {
        bound = len(p)
    } else if x <= len(p) {
        bound = x
    }

    buf := make([]byte, bound)
    for n < bound {
        if char := alpha(a.src[a.cur]); char != 0 {
            buf[n] = char
        }
        n++
        a.cur++
    }
    copy(p, buf)
    return n, nil
}
```

