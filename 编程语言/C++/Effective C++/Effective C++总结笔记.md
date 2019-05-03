## Effective C++阅读总结

### 以const,enum,inline替换#define(原则02)
预编译时的信息不会被带到符号表

 - 对于单纯常量,最好以const对象或enums替换#defines.
 - 对于形似函数的宏(macros) ,最好改用inline函数替换#defines。

### 尽可能用const

![](image/eff0.png)