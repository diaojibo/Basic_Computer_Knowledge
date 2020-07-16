## Go与C++的交互



### Go程序编译成So

首先我们要解决的问题是Go程序编译成So

看例程：

```go
package main

import (
        "C"
        "image"
        _ "image/jpeg"
        _ "image/png"
        "os"
)

//export ImgutilGetImageSize
func ImgutilGetImageSize(path *C.char, w *uint, h *uint) *C.char {
        file, err := os.Open(C.GoString(path))
        if err != nil {
                return C.CString(err.Error())
        }
        defer file.Close()

        img, _, err := image.Decode(file)
        if err != nil {
                return C.CString(err.Error())
        }

        rect := img.Bounds()
        *w = uint(rect.Dx())
        *h = uint(rect.Dy())

        return nil
}

func main() {}
```

首先要保证：

- go程序要import "C"库
- 导出的函数一定要在“main”这个包中
- 要导出的函数里，需要有一个export注释
- 函数签名不能包含Go结构体、Go interface、Go数组或可变参数。

通过以下命令来生成so

```
go build -buildmode c-shared -o ./build/go_main.so
```

