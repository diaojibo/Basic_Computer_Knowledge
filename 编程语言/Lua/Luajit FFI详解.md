## Luajit FFI
对 C 语言良好的亲和力，一直是 Lua 的优势之一。LuaJIT 在传统的 Lua C API 之外，额外提供 FFI 的方式来调用 C 函数，更是大大提升了跟 C 交互的便利度。

甚至有这么一种说法，虽然 LuaJIT 命名是 Lua + JIT，但是好多人是冲着 FFI 去用 LuaJIT 的。

**FFI 全称是 Foreign Function Interface**，即一种在 A 语言中调用 B 语言的机制。通常来说，指其他语言调用 C 的函数。
既然是跨语言调用，就必须解决 C 函数查找和加载，以及 Lua 和 C 之间的类型转换的问题。

当然，我们也可以用C API的方式来让Lua进行C函数的调用，但无疑，FFI就是Luajit的一大优势。

**总之，luajit ffi 是 luajit 提供给 Luaer 使用 Lua 调用 C 函数的 Lua 库，使用该库，Luaer 不用再去操作复杂的 Lua 栈来粘合两种程序代码**

### Lua引入luajit ffi库
lua引用ffi库其实比较简单：

``` lua
local ffi = require("ffi")
```

### FFI的使用
FFI使用步骤其实就分为三步：

1. 加载FFI模块

2. 使用ffi.cdef声明C函数或者C的数据结构

3. 使用ffi模块其他函数

#### FFI库一些术语
 - cdecl:一个抽象的C类型定义(其实是一个lua字符串)
 - ctype:一个C类型对象
 - cdata:一个C数据对象
 - ct:一个C类型格式，就是一个模板对象，可能是cdecl,cdata,ctype
 - cb:一个回调对象
 - VLA:一个可变长度的数组
 - VLS:一个可变长度的结构体


#### ffi.cdef
利用ffi.cdef可以声明C函数或C数据结构

``` lua
-- 定义一个结构体
ffi.cdef([[
    typedef struct Student {
        char no[50];
        char name[50];
    } Stu;
]])

-- 定义一个系统函数
ffi.cdef([[
    int printf(const char *fmt,...);
]])

-- 定义一个第三方函数
ffi.cdef([[
    long filesize(const char *filename);
]])

```

如果是系统函数，我们可以直接用ffi.C去调用了。但是必须是必须是先声明再使用。

``` lua
ffi.C.printf("hello world!\n")
```

如果是调用第三方(动态)函数库【也就是so里那种】：

``` lua
local ffi = require 'ffi'
ffi.cdef[[
    void *curl_easy_init();
    int curl_easy_setopt(void *curl, int option, ...);
    int curl_easy_perform(void *curl);
    void curl_easy_cleanup(void *curl);
    char *curl_easy_strerror(int code);
]]
local libcurl = ffi.load('curl')
local curl = libcurl.curl_easy_init()
local CURLOPT_URL = 10002                -- 参考 curl/curl.h 中定义
if curl then
    libcurl.curl_easy_setopt(curl, CURLOPT_URL, 'http://www.baidu.com')
    res = libcurl.curl_easy_perform(curl)
    if res ~= 0 then
        print(ffi.string(libcurl.curl_easy_strerror(res)))
    end
    libcurl.curl_easy_cleanup(curl)
end
```

### ffi.string
从char数组中获取lua string

```
str = ffi.string(req.body_)
```