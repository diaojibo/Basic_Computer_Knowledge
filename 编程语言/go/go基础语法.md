## go基础语法
go的语言符号有5种

1. 标识符identifer，其实就是变量或者函数声明的命名。开头大写说明允许外部访问，开头小写说明私有。

2. 关键字keyword，其实就是保留字

3. 字面量literal，可以理解为赋值语句中右边的那部分，比如字符串啊，数值之类的。

4. 分隔符delimiter

5. 操作符operator

### 数据类型
数据类型的出现是为了把数据分成所需内存大小不同的数据，编程的时候需要用大数据的时候才需要申请大内存，就可以充分利用内存。

大类，分为：

1. 数字类型
2. 布尔型
3. 字符串类型
4. 派生类型

#### 数字类型
Go 也有基于架构的类型，例如：int、uint 和 uintptr。跟C++很像

比如整形就有

![](image/go0.png)

浮点型则有：
 - float32
 - float64
 - complex64 (32位实数和虚数)
 - complex128 (64位实数和虚数)

其余的数字类型还有

 - byte
 - rune
 - uint
 - int
 - uintptr(无符号整形，用来存放指针)

除此之外，go还内置一些高级类型，比如自定义结构体struct，集合map等。

### 变量声明
三种定义方法：

``` go
// 例如
var a int = 10
var b = 10
c := 10
```

第一种变量名后面要跟类型，第二种通过赋初值自动判断，第三种用了冒号等号语法糖。

除此之外还有多变量声明：

``` go
//类型相同多个变量, 非全局变量
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 //和python很像,不需要显示声明类型，自动推断

vname1, vname2, vname3 := v1, v2, v3 //出现在:=左侧的变量不应该是已经被声明过的，否则会导致编译错误


// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)
```

#### 值类型和引用类型
go中当然也有这个概念，基础数据类型都是值类型。复杂的类型就是引用类型。

### 常量
常量是一个简单值的标识符，在程序运行时，不会被修改的量。

**常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。**

``` go
const LENGTH int = 10
const WIDTH int = 5   
var area int
const a, b, c = 1, false, "str" //多重赋值
```

#### iota
iota，特殊常量，可以认为是一个可以被编译器修改的常量。

**在每一个const关键字出现时，被重置为0，然后再下一个const出现之前，每出现一次iota，其所代表的数字会自动增加1。**

iota 可以被用作枚举值：

``` go
const (
    a = iota
    b = iota
    c = iota
)
```

所以 a=0, b=1, c=2

再看一个复杂点的

``` go
const (
          a = iota   //0
          b          //1
          c          //2
          d = "ha"   //独立值，iota += 1
          e          //"ha"   iota += 1
          f = 100    //iota +=1
          g          //100  iota +=1
          h = iota   //7,恢复计数
          i          //8
  )
```

### 多返回值函数
函数可以返回多个值

``` go
package main

import "fmt"

func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Mahesh", "Kumar")
   fmt.Println(a, b)
}
```

**go语言的参数传递使用的是值传递。**

#### 函数作为值

``` go
/* 声明函数变量 */
getSquareRoot := func(x float64) float64 {
   return math.Sqrt(x)
}
```

#### 闭包
Go 语言支持匿名函数，可作为闭包。匿名函数是一个"内联"语句或表达式。匿名函数的优越性在于可以直接使用函数内的变量，不必申明。

以下实例中，我们创建了函数 getSequence() ，返回另外一个函数。该函数的目的是在闭包中递增 i 变量，代码如下：

``` go
package main

import "fmt"

func getSequence() func() int {
   i:=0
   return func() int {
      i+=1
     return i  
   }
}

func main(){
   /* nextNumber 为一个函数，函数 i 为 0 */
   nextNumber := getSequence()  

   /* 调用 nextNumber 函数，i 变量自增 1 并返回 */
   fmt.Println(nextNumber())
   fmt.Println(nextNumber())
   fmt.Println(nextNumber())

   /* 创建新的函数 nextNumber1，并查看结果 */
   nextNumber1 := getSequence()  
   fmt.Println(nextNumber1())
   fmt.Println(nextNumber1())
}
```

### go指针
go跟C++一样也是有指针的。

``` go
var a int= 20   /* 声明实际变量 */
var ip *int        /* 声明指针变量 */

ip = &a  /* 指针变量的存储地址 */
```

当一个指针被定义后没有分配到任何变量时，它的值为 nil。

nil 指针也称为空指针。

nil在概念上和其它语言的null、None、nil、NULL一样，都指代零值或空值。

``` go
if(ptr != nil)     /* ptr 不是空指针 */
if(ptr == nil)    /* ptr 是空指针 */
```

### Map(集合)&Slice(切片)
Map 是一种无序的键值对的集合。

``` go
countryCapitalMap [ "France" ] = "Paris"
```

Slice切片是对数组的抽象，本质其实就是一个动态数组，类似于js或者python中的list。

切片不需要说明长度。
或使用make()函数来创建切片

``` go
var slice1 []type = make([]type, len)

也可以简写为

slice1 := make([]type, len)
```

#### 数组

讲到切片slice，不得不再讲讲go里的数组，go里也是有数组这个概念的，类型定长的列表就是数组。**数组是具有相同唯一类型的一组已编号且长度固定的数据项序列**

```go
var balance [10] float32
```

上面就定义了一个长度为10的数组，如果不定长，那就是slice切片。

#### 切片

我们还不够理解切片，切片不是像C语言一样，是一个数组头部的指针，并不是这样的。

slice在go里的结构体如下 reflect.SliceHeader：

```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

![1593660498533](image/1593660498533.png)

这下就很清楚了，切片也不过是个结构体，有三个元素 ptr len cap。所以当我们**函数传递切片**的时候，传递的就是这个header(只是复制切片头信息（`reflect.SliceHeader`）)，底层的数组是一致的。**所以放心大胆的去传递切片，不会有冗余拷贝的问题**。



### range

Go 语言中 range 关键字用于 for 循环中迭代数组(array)、切片(slice)、通道(channel)或集合(map)的元素。

**在数组和切片中它返回元素的索引和索引对应的值，在集合中返回 key-value 对的 key 值。**

``` go
//这是我们使用range去求一个slice的和。使用数组跟这个很类似
nums := []int{2, 3, 4}
sum := 0
for _, num := range nums {
    sum += num
}
fmt.Println("sum:", sum)
```

### 内建函数make
内建函数make(T, args)与new(T)的用途不一样。**它只用来创建slice，map和channel，并且返回一个初始化的(而不是置零)，类型为T的值（而不是*T）**

所以map,Slice,channel经常用make函数来创建。

### struct和函数方法
go没有class这个关键字。实现面向对象一般都是用struct结构体。而想定义成员方法，那就通过在定义函数时加关键字实现。

``` go
/* 定义结构体 */
type Circle struct {
  radius float64
}

//该 method 属于 Circle 类型对象中的方法
func (c Circle) getArea() float64 {
  //c.radius 即为 Circle 类型对象中的属性
  return 3.14 * c.radius * c.radius
}
```

上面的描述还有些欠缺，再来理一理成员方法，也就是所谓的**方法接收器**

在Go语言中，将函数绑定到具体的类型中，则称该函数是该类型的方法，其定义的方式是在func与函数名称之间加上具体类型变量，这个类型变量称为`方法接收器`

```go
func setName(m Member,name string){//普通函数
    m.Name = name
}

func (m Member)setName(name string){//绑定到Member结构体的方法
    m.Name = name
}

```

通过`方法接收器`可以访问结构体的字段，这类似其他编程语言中的this关键词，但在Go语言中，只是一个变量名而已，我们可以任意命名`方法接收器`。

```go
m := Member{}
m.setName("小明")
fmt.Println(m.Name)//输出为空
```

上面的代码中，我们会很奇怪，不是调用setName()方法设置了字段Name的值了吗？为什么还是输出为空呢？

这是因为，**结构体是值传递**，当我们调用setName时，方法接收器接收到是只是结构体变量的一个副本，通过副本对值进行修复，并不会影响调用者，因此，**我们可以将方法接收器定义为指针变量，就可达到修改结构体的目的了**。

```go
func (m *Member)setName(name string){/将Member改为*Member
    m.Name = name
}

m := new(Member)
m.setName("小明")
fmt.Println(m.Name)//小明
```



### 接口

go里的接口也比较独特，它把所有的具有共性的方法定义在一起，**任何其他类型只要实现了这些方法就是实现了这个接口。**

不需要你显示区implement这个接口

``` go
type Phone interface {
    call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
    fmt.Println("I am Nokia, I can call you!")
}
```

如上定义后，nokiaPhone类型即可用phone接口定义使用。

``` go
var phone Phone
phone = new(NokiaPhone)
phone.call()
```
