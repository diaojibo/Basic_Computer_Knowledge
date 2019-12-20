## 仿函数

仿函数(functor)，就是使一个类的使用看上去象一个函数。其实现就是类中实现一个operator()，这个类就有了类似函数的行为，就是一个仿函数类了。

``` c++
class FunctorType
{
public:
    void operator() ()
    {
        //...
    }
};
```

在调用的时候可以：

``` c++
FunctorType fo;
fo();

```

仿函数相比普通函数也是有优势的：

 -  仿函数比普通函数更灵活。它可以有很多“状态”,这是因为仿函数是在类的内部实现的，类中的成员变量可以表示对象的不同状态，这就使得仿函数有了不同的状态。
 - 每个仿函数都有自己的型别。因此可以将仿函数的型别当做template参数来传递，从而指定某种行为模式。容器型别也会因为仿函数的不同而不同。
 - 在执行的速度上，仿函数通常比函数指针要快。


### std::function
类模板 std::function(头文件是functional) 是 **通用多态函数封装器**。 std::function 的实例能存储、复制及调用任何可调用 (Callable) 目标——**函数、 lambda 表达式、 bind 表达式或其他函数对象，还有指向成员函数指针和指向数据成员指针**，c++11中非常便利的类型。

存储的可调用对象被称为 std::function 的目标。若 std::function 不含目标，则称它为空。调用空 std::function 的目标导致抛出 std::bad_function_call 异常。

事实上就是一个可调用目标的容器，用来封装函数针，lambda等。封装完成后就是一个伪函数对象，可以直接调用。

### bind
bind是这样一种机制，它可以预先把指定可调用实体的某些参数绑定到已有的变量，产生一个新的可调用实体，这种机制在回调函数的使用过程中也颇为有用。

zwlj:也就是可以根据使某些函数的某些参数固定从而生成一种新的函数，有点像函数式编程的思想。

``` c++
int Func(int x, int y);  
auto bf1 = std::bind(Func, 10, std::placeholders::_1);  
bf1(20); ///< same as Func(10, 20)  
```

如上代码我们把第0个参数绑定成了10。


万分注意，当我们对类的成员函数使用bind的时候，要注意成员函数的第0个参数是this，也就是实例本身，所以我们要这样用：

``` c++
class A  
{  
public:  
    int Func(int x, int y);  
};  

A a;  

std::function< int(int)> bf3 = std::bind(&A::Func, a, std::placeholders::_1, 100);  
bf3(10); ///< same as a.Func(10, 100)  
```

上面bf3，就是定义一个伪函数function容器bf3，并用bind func来初始化，绑定了实例对象a的成员函数Func，切把第二个参数int也固定了。

