## interface详解

**Go 中的 interface 是一种类型**，更准确的说是一种抽象类型 abstract type，一个 interface 就是包含了一系列行为的 method 集合

```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

Interface类型可以定义**一组方法**，但是这些不需要实现。请注意：此处限定是**一组方法**，既然是方法，就不能是变量；而且是**一组**，表明可以有多个方法。再多声明一点，**interface本质上是一种类型**，确切的说，是**指针类型**

如果接口没有任何方法声明，**那么就是一个空接口（interface{}）**，它的用途类似面向对象里的根类型Object，可被赋值为任何类型的对象。接口变量默认值是nil。如果实现接口的类型支持，可做相等运算。

```go
func main() { 
   var t1,t2 interface{} 
   println(t1==nil,t1==t2) 
  
   t1,t2=100,100
   println(t1==t2) 
  
   t1,t2=map[string]int{},map[string]int{} 
   println(t1==t2) 
}
```

则有输出

```
true true
true
panic:runtime error:comparing uncomparable type map[string]int
```



### 类型推断

类型推断可将接口变量还原为原始类型，或用来判断是否实现了某个更具体的接口类型。

```go
type data int
  
func(d data)String()string{ 
   return fmt.Sprintf("data:%d",d) 
} 
  
func main() { 
   var d data=15
   var x interface{} =d
  
   if n,ok:=x.(fmt.Stringer);ok{  // 转换为更具体的接口类型 
       fmt.Println(n) 
    } 
  
   if d2,ok:=x.(data);ok{        // 转换回原始类型 
       fmt.Println(d2) 
    } 
  
   e:=x.(error)           // 错误:main.data is not error
   fmt.Println(e) 
}
```

### 实现多态

多态功能是interface实现的重要功能，也是Golang中的一大行为特色，其多态功能一般要结合Go method实现，作为函数参数可以容易的实现多态功能。

```go
package main

import "fmt"

// notifier是一个定义了通知类行为的接口
type notifier interface {
　 notify()
}

// 定义user及user.notify方法
type user struct {
　 name　string
　 email string
}

func (u *user) notify() {
　 fmt.Printf("Sending user email to %s<%s>\n",
　　　 u.name,
　　　 u.email)
}

// 定义admin及admin.notify方法
type admin struct {
　 name　string
　 email string
}

func (a *admin) notify() {
　 fmt.Printf("Sending admin email to %s<%s>\n",
　　　 a.name,
　　　 a.email)
}

func main() {
　 // 创建一个user值并传给sendNotification
　 bill := user{"Bill", "bill@email.com"}
　 sendNotification(&bill)

　 // 创建一个admin值并传给sendNotification
　 lisa := admin{"Lisa", "lisa@email.com"}
　 sendNotification(&lisa)
}

// sendNotification接受一个实现了notifier接口的值
// 并发送通知
func sendNotification(n notifier) {
　 n.notify()
}
```

上述代码中实现了一个多态的例子，函数`sendNotification`接受一个实现了notifier接口的值作为参数。既然任意一个实体类型都能实现该接口，那么这个函数可以针对任意实体类型的值来执行`notify`方法，调用`notify`时，会根据对象的实际定义来实现不同的行为，从而实现多态行为。

