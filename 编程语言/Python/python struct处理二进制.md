## Python struct处理二进制

有的时候需要用python处理二进制数据，比如，存取文件，socket操作时.这时候，可以使用python的struct模块来完成.可以用 struct来处理c语言中的结构体.

struct模块中最重要的三个函数是pack(), unpack(), calcsize()

- pack(fmt, v1, v2, ...)     按照给定的格式(fmt)，把数据封装成字符串(实际上是类似于c结构体的字节流)

- unpack(fmt, string)       按照给定的格式(fmt)解析字节流string，返回解析出来的tuple
- calcsize(fmt)                 计算给定的格式(fmt)占用多少字节的内存

struct中支持的格式如下表：

| Format | C Type               | Python             | 字节数 |
| ------ | -------------------- | ------------------ | ------ |
| `x`    | pad byte             | no value           | 1      |
| `c`    | `char`               | string of length 1 | 1      |
| `b`    | `signed char`        | integer            | 1      |
| `B`    | `unsigned char`      | integer            | 1      |
| `?`    | `_Bool`              | bool               | 1      |
| `h`    | `short`              | integer            | 2      |
| `H`    | `unsigned short`     | integer            | 2      |
| `i`    | `int`                | integer            | 4      |
| `I`    | `unsigned int`       | integer or long    | 4      |
| `l`    | `long`               | integer            | 4      |
| `L`    | `unsigned long`      | long               | 4      |
| `q`    | `long long`          | long               | 8      |
| `Q`    | `unsigned long long` | long               | 8      |
| `f`    | `float`              | float              | 4      |
| `d`    | `double`             | float              | 8      |
| `s`    | `char[]`             | string             | 1      |
| `p`    | `char[]`             | string             | 1      |
| `P`    | `void *`             |                    |        |

为了同c中的结构体交换数据，还要考虑有的c或c++编译器使用了字节对齐，通常是以4个字节为单位的32位系统，故而struct根据本地机器字节顺序转换.可以用格式中的第一个字符来改变对齐方式.定义如下：

| Character | Byte order             | Size and alignment            |
| --------- | ---------------------- | ----------------------------- |
| `@`       | native                 | native            凑够4个字节 |
| `=`       | native                 | standard        按原字节数    |
| `<`       | little-endian          | standard        按原字节数    |
| `>`       | big-endian             | standard       按原字节数     |
| `!`       | network (= big-endian) | standard       按原字节数     |

使用方法是放在fmt的第一个位置，就像'@5s6sif'

### 示例

比如有一个结构体

```c++
struct Header

{

    unsigned short id;

    char[4] tag;

    unsigned int version;

    unsigned int count;

}
```

通过socket.recv接收到了一个上面的结构体数据，存在字符串s中，现在需要把它解析出来，可以使用unpack()函数.

```python
import struct

id, tag, version, count = struct.unpack("!H4s2I", s)
```

！代表网络序，H代表短整形，4s对应4个char的文职，2I对应两个无符号整形