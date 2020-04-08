## defer

go 的 defer 语句是用来**延迟执行函数的**，而且延迟发生在**调用函数 return 之后**

```go
func a() int {
  defer b()
  return 0
}
```

b 的执行是发生在 return 0 之后，注意 defer 的语法，关键字 defer 之后是函数的调用。



### defer特性

defer 的作用就是把关键字之后的函数执行压入一个栈中延迟执行，多个 defer 的执行顺序是后进先出 LIFO ：

```go
defer func() { fmt.Println("1") }()
defer func() { fmt.Println("2") }()
defer func() { fmt.Println("3") }()
```

输出顺序是 321。

这个特性可以对一个 array 实现逆序操作。



一个函数被 defer 时，它的参数在 defer 时进行计算确定，即使 defer 之后参数发生修改，对已经 defer 的函数没有影响，什么意思？看例子：

```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```

a 执行输出的是 0 而不是 1，因为 defer 时，i 的值是 0，此时被 defer 的函数参数已经进行执行计算并确定了。



### 清理资源

由于 defer 的延迟特性，defer 常用在函数调用结束之后清理相关的资源，比如

```go
f, _ := os.Open(filename)
defer f.Close()
```

文件资源的释放会在函数调用结束之后借助 defer 自动执行，不需要时刻记住哪里的资源需要释放，打开和释放必须相对应。

用一个例子深刻诠释一下 defer 带来的便利和简洁。

代码的主要目的是打开一个文件，然后复制内容到另一个新的文件中，没有 defer 时这样写：

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil { //1
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

代码在 `#1` 处返回之后，src 文件没有执行关闭操作，可能会导致资源不能正确释放，改用 defer 实现：

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

src 和 dst 都能及时清理和释放，无论 return 在什么地方执行。

鉴于 defer 的这种作用，defer 常用来释放数据库连接，文件打开句柄等释放资源的操作。



### 执行recover

被 defer 的函数在 return 之后执行，这个时机点正好可以捕获函数抛出的 panic，因而 defer 的另一个重要用途就是执行 recover。

recover 只有在 defer 中使用才更有意义，如果在其他地方使用，由于 program 已经调用结束而提前返回而无法有效捕捉错误。

```
package main

import (
    "fmt"
)

func main() {
    defer func() {
        if ok := recover(); ok != nil {
            fmt.Println("recover")
        }
    }()

    panic("error")

}
```

记住 defer 要放在 panic 执行之前。