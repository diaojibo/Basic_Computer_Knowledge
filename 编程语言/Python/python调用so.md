## python调用so
python想要调用C++编写的so，有多方面的准备。

首先当然是这个so相关的要求

python调用so，无非就是根据符号表找到二进制文件中对应的函数来进行调用。

### c++函数要用extern C包装
因为python调用so的方式是根据C风格来寻址，所以必须用extern "C"来进行包装。

``` c++
extern "C" {
    Foo* Foo_new(){ return new Foo(); }
    void Foo_bar(Foo* foo){ foo->bar(); }
}

extern "C" int test_call(int testint)
{
    printf("hello pkg send tool %d\n ", testint);

    return 999;
}

```

全局函数，这样保证python能找到

### python调用方式
python直接调用ctypes这玩意就好了

``` python
test = cdll.LoadLibrary("./libpkg_send_tool.so")

r = 100
ret = test.test_call(r)
```

这样就能顺利返回一个整数了。

### 相互调用字符串
拿假如python想调用一个字符串然后获得一个字符串呢

可以这样进行编码：

假如有以下一段c++编码

``` c++
extern "C" const char* test_str_call(const char* st)
{

    char output_buffer[1024];
    snprintf(output_buffer,sizeof(output_buffer),"hello pkg send test str %s \n", st);

    return output_buffer;
}
```

python可以这样进行调用，注意，ctypes默认函数执行后的返回值是int，所以一定要显示指定，其返回的其实是一个c类型字符串：

``` python
test.test_str_call.restype = c_char_p
strtt = "niubi"
retstr = test.test_str_call(strtt)

print(retstr)
```

#### 安全调用
其实上面的方法并不安全，因为返回的时候，output_buffer里的内容就会被回收掉了。其实不稳定，我们还是需要创建一个字符串buffer然后传递进去的：

```
t = ctypes.create_string_buffer(100)
print(ctypes.sizeof(t))

ttt = test.test_str_call(t)

print(t.value)
```