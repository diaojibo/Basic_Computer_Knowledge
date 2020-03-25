## go语言基础

### 特性
go语言的特性可以很好的概括：

 - 开放源码
 - 静态类型和编译型(声明时指定类型，而且要编译成可执行文件方可执行)
 - 跨平台(跨计算架构和操作系统)
 - 自动垃圾回收
 - 原生并发编程
 - 多编程范式
 - **代码风格强制统一**

### 工程结构
go 是一个推崇软件工程理念的编程语言，所以有几个概念需要理清。

#### 工作区
go项目源码的目录，就是工作区。

一般来说，可以构建三个子目录，src，pkg和bin

 - src目录，源码目录，而且相关模块的代码要在相关命名的目录下。
 - pkg目录，通过go install安装后的代码包的归档文件。
 - bin目录，通过go install完成安装后，代码源码生成的可执行文件。

只看上面的划分可能还不明确，实质上，go项目喜欢这样分，所有的Go项目统一放到一个Go Project文件夹

```
C:\Go    <<--- GOROOT 指向的位置 //linux 下一般为/usr/local/go
    --src                 <<--- Go 语言自带的源代码
    --pkg                 <<--- 编译的中间文件放在此文件夹
    --bin                 <<--- 编译的目标文件放在此文件夹
D:\MyWorks  <<--- GOPATH 指向的位置
    --src                 <<--- 项目源代码放置在此文件夹。!!!警告：一个常犯的错误是把 GOPATH 指向此处!!!
        --HelloWorld      <<--- 我们项目源代码所在的文件夹。!!!警告：一个常犯的错误是把 GOPATH 指向此处!!!
        --vendor          <<--- 第三方开源代码文件夹
            --github.com
                --...
    --pkg                 <<--- 编译的中间文件放在此文件夹，Go编译器自动生成此文件夹
    --bin                 <<--- 编译的目标文件放在此文件夹，Go编译器自动生成此文件夹
```

**代码文件开头第一行要声明package**,一般同一个目录下的源码文件要属于同一个package才行。

#### GOPATH
go项目所在的目录，也就是刚才提到的src目录的上一层。

新版的go安装之时间，gopath会被自动设置在~/go下

#### 命令源码文件
我们想要执行go文件，那么这个go文件就必须是 **命令源码文件**。命令源码文件需要有一个main函数，而且这个文件必须属于package main，否则会不能执行

``` go
package main
func main(){
  ...
}

```

由于刚才提到的同一目录下要同属同一个package的原则，所以命令源码文件要单独开一个目录(代码包)。

zwlj：也就是命令源码文件必须属于main包

#### 安装依赖
在下载一个第三方的go项目的时候，我们直接go build还不一定可以，要安装它的依赖。通过直接在项目下：

```
go get
```


#### go install&build
执行go build会在当前目录生成一个和包名一致的可执行文件，go install则会在bin目录下产生。

**此时注意，我们在import引入包的时候，其实是以GOPATH为根目录的**

比如你可以把你的可执行(项目)包，安放在某个 $GOPATH/src下，例如 $GOPATH/src/app/youpackagedir

这样本地包的import就变成

`import "app/yourpackagedir/subpackage"`



### go mod replace

本地项目查找依赖的时候，我们经常要在mod文件里这么干

```go
module github.com/EDDYCJY/go-gin-example

go 1.13

require (...)

replace (
		pkg/setting => ./pkg/setting
)
```

通过这样把某些包的查找路径链接到本地，这样可以跳转。

