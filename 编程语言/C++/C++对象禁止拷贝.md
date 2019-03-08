## C++ 对象禁止拷贝
C++有时候需要禁止对象的拷贝。比如，房地产代理商出售房屋，服务于这样的代理商的软件系统自然要有一个 class（类）来表示被出售的房屋：

``` c++
class HomeForSale { ... };
```

每一个房地产代理商都会很快指出，每一件房产都是独特的——没有两件是完全一样的。在这种情况下，为HomeForSale object（对象）做一个 copy（拷贝）的想法就令人不解了。你怎么能拷贝一个独一无二的东西呢？因此最好让类似这种企图拷贝 **HomeForSale object（对象）的行为不能通过编译**

老式写法：

``` c++
class noncopyable  
{  
protected:  
    noncopyable() {}  
    ~noncopyable() {}  
private:  
    noncopyable( const noncopyable& );  
    noncopyable& operator=( const noncopyable& );  
};
```

C++11 写法：

``` c++
class noncopyable  
{  
protected:  
    //constexpr noncopyable() = default;  
   // ~noncopyable() = default;  
    noncopyable( const noncopyable& ) = delete;  
    noncopyable& operator=( const noncopyable& ) = delete;  

```