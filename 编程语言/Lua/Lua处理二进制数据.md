## Lua处理二进制数据

lua在处理通信的时候,总会设计到数据处理,例如:

```
socket:send
socket:recv
spi.send
spi.recv
uart.write
```

这些方法要么发送的string,要么返回string, 而不是使用其他编程语言的byte[]/char[], 因为lua没有byte和char

### 二进制处理

对于二进制数据，比如有2byte的数据，用16进制表示成0x3132

已知Ascill码 0x31位char “1” 0x32为char“2”，那么

- 0x3132 --此乃数值标识
- "3132" 这个16进制数写成string给人看
- "12" 0x3132的等价表示，0x3132转成字符串就是“12”
- ｛0x31,0x32｝lua数组table
- [0x31,0x32] 其他语言的byte数组

#### 数据转换

用string.char逐个字节描述

```
local data = string.char(0x31,0x32)
```

又或者用string.fromHex来处理,这里需要传入2的倍数Hex

```
local data = string.fromHex("3132")
```

