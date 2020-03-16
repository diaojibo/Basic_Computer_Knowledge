## init函数

大家都知道golang里的main函数是程序的入口函数，main函数返回后，程序也就结束了。golang还有另外一个特殊的函数init函数，先于main函数执行，实现包级别的一些初始化操作，今天我们就深入介绍下init的特性。



golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：

1. 初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似，参见[golang变量的初始化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
2. 初始化包作用域的变量（该作用域的变量的初始化也并非按照“从上到下、从左到右”的顺序，runtime解析变量依赖关系，没有依赖的变量最先初始化，参见[golang变量的初始化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
3. 执行包的init函数；



比如有这几个文件

**pack.go**:

```go
package pack                                                                                                                     

import (
   "fmt"
   "test_util"
)

var Pack int = 6               

func init() {
   a := test_util.Util        
   fmt.Println("init pack ", a)    
} 
```

**test_util.go**

```go
package test_util                                                                                                                

import "fmt"

var Util int = 5

func init() {
   fmt.Println("init test_util")
}  
```

**main.go**

```go
package main                                                                                                                     

import (
   "fmt"
   "pack"
   "test_util"                
)

func main() {                  
   fmt.Println(pack.Pack)     
   fmt.Println(test_util.Util)
}
```

输出：

```text
init test_util
init pack  5
6
5
```

**由于pack包的初始化依赖test_util，因此运行时先初始化test_util再初始化pack包；**