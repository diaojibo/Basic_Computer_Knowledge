## C API

其实我们用Lua和C系交互，怎么也绕不过C API，无论你是标准Lua还是Luajit。交互的是C还是C++, C API是C代码和Lua进行交互的函数集。

主要分为以下一些构成：

 - 读写Lua全局变量的函数
 - 调用Lua函数的函数
 - 运行Lua代码片断的函数
 - 注册C函数然后可以在Lua中被调用的函数

等等。

当在Lua和C之间交换数据时我们面临着两个问题：**动态与静态类型系统的不匹配和自动与手动内存管理的不一致**。解决办法是**在C和Lua之间通信关键在于一个虚拟的栈**。几乎所有的API调用都是对栈上的值进行操作，所有C与Lua之间的数据交换也都通过这个栈来完成。因为栈是由Lua来管理的，垃圾回收器知道那个值正在被C使用。

zwlj：Lua和C通过一个虚拟栈(an abstract stack)来进行数据传递，栈的数据由Lua虚拟机来管理。

因此对于这个栈：

1. 当你调用Lua时，它只会改变栈顶部分。
2. 你的Ｃ代码却有更多的自由；更明确的来讲，你可以查询栈上的任何元素，甚至是在任何一个位置插入和删除元素。

### C API的头文件
在调用C API时有几个重要的头文件：

1. lua.h：Lua基础函数库，lua_前缀
2. lauxlib.h：辅助库，luaL_前缀，利用lua.h实现的更高层的抽象
3. lualib.h：为了保持Lua的苗条，所有的标准库以单独的包提供，所以如果你不需要就不会强求你使用它们。头文件lualib.h定义了打开这些库的函数。例如，调用luaopen_io，以创建io table并注册I/O函数（io.read,io.write等等）到Lua环境中。

如果是Luajit，当然还会有Luajit的头文件。但是Luajit最后也是对C API的扩展。所以这里不叙述。


### State的初始化
Lua Library是可重用的，意为，Lua Library设计的时候就是没有全局变量的。当我们用C去调用Lua脚本的时候，Lua整个解释器的状态(全局变量，堆栈)，都会存储在一个`lua_state`的动态结构当中。开始时通过调用 `lua_newstate()`进行初始化。

所以就不难理解我们的所有操作，都需要传递一个Lua_state指针。且最后我们要调用lua_close去销毁释放掉这个state。

有了这个state我们就可以通过刚才我们说的虚拟栈，去与lua交互了。


zwlj：这个lua_state我觉得可以理解为lua虚拟机的内存，C从外部可以通过访问这个state获取到想要的数据。


### api

关于更细致的api介绍，只能去阅读官方文档：https://www.lua.org/manual/5.0/manual.html#3.1

#### lua_push*：压入栈元素

API用索引来访问栈中的元素。栈底为1，依次往上增加，也可用负数索引，-1表示栈顶元素。下面列出一些常用的C API函数：

```
void lua_pushnil (lua_State *L);
void lua_pushboolean (lua_State *L, int bool);
void lua_pushnumber (lua_State *L, double n);
void lua_pushlstring (lua_State *L, const char *s, size_t length);
void lua_pushstring (lua_State *L, const char *s);
```

#### lua_to*：从栈中获得值
即使给定的元素类型不正确，调用这些函数也没问题

```
int lua_toboolean (lua_State *L, int index);
double lua_tonumber (lua_State *L, int index);
const char * lua_tostring (lua_State *L, int index);
```

Lua_tostring函数返回一个指向字符串的内部拷贝的指针。你不能修改它（使你想起那里有一个const）。只要这个指针对应的值还在栈内，Lua会保证这个指针一直有效。当一个C函数返回后，Lua会清理他的栈，所以，有一个原则：**永远不要将指向Lua字符串的指针保存到访问他们的外部函数中**。

#### 其他api

```
size_t lua_strlen (lua_State *L, int index)：返回字符串的实际长度。

int lua_checkstack(lua_State *L, int sz)：检查栈空间。默认有20个空闲的记录，lua.h中的LUA_MINSTACK宏定义了这个常量。

int lua_is... (lua_State *L, int index)：检查一个元素能否被转换成指定的类型。

int lua_type (lua_State *L, int idx)：返回栈中元素的类型；

const char* lua_typename(lua_State *L, int tp)：返回type对应的名字字符串，第二个参数为lua_type返回的类型

void luaL_checktype (lua_State *L, int arg, int t)：返回参数arg是否是类型t，第三个参数为lua_type的取值。
　　
在lua.h头文件中，每种类型都被定义为一个常量：LUA_TNIL、LUA_TBOOLEAN、LUA_TNUMBER、LUA_TSTRING、LUA_TTABLE、LUA_TFUNCTION、LUA_TUSERDATA以及LUA_TTHREAD。

int lua_gettop (lua_State *L)：返回栈中元素个数，它也是栈顶元素的索引。

void lua_settop (lua_State *L, int index)：设置栈顶元素的索引，相当于设置栈的大小。如果开始的栈顶高于新的栈顶，顶部的值被丢弃。否则，为了得到指定的大小这个函数压入相应个数的空值（nil）到栈上。lua_settop(L,0)：清空堆栈。

#define lua_pop(L,n) lua_settop(L, -(n)-1)：宏定义，弹出n个元素。

void lua_pushvalue (lua_State *L, int index)：压入堆栈上指定索引的一个抟贝到栈顶，等于拷贝index处的元素，然后添加到栈顶。

void lua_remove (lua_State *L, int index)：移除指定索引的元素，并将其上面所有的元素下移来填补这个位置的空白。

void lua_insert (lua_State *L, int index)：移动栈顶元素到指定索引的位置，并将这个索引位置上面的元素全部上移至栈顶被移动留下的空隔。

void lua_replace (lua_State *L, int index)：从栈顶弹出元素值并将其设置到指定索引位置，没有任何移动操作。
```

### Lua调用C的函数

C和Lua进行函数交互时，设定的函数都需要遵循一些约定，比如获取时要先获取参数的个数。return的时候也要return返回值的个数。比如下面定义的C函数。


``` c++
static int foo (lua_State *L) {
         int n = lua_gettop(L);    /* number of arguments */
         lua_Number sum = 0;
         int i;
         for (i = 1; i <= n; i++) {
           if (!lua_isnumber(L, i)) {
             lua_pushstring(L, "incorrect argument to function `average'");
             lua_error(L);
           }
           sum += lua_tonumber(L, i);
         }
         lua_pushnumber(L, sum/n);        /* first result */
         lua_pushnumber(L, sum);         /* second result */
         return 2;                   /* number of results */
       }
```

然后我们可以用官方给的一个宏来注册这个函数给Lua，具体来说就是把这个函数 通过虚拟栈push 给 Lua

``` c++
#define lua_register(L,n,f) \
               (lua_pushstring(L, n), \
                lua_pushcfunction(L, f), \
                lua_settable(L, LUA_GLOBALSINDEX))
```

然后我们调用：
 
``` c++
lua_register(L, "average", foo);
```

在lua脚本中的时候，我们就可以调用此函数了，但是要注意，我们不需要传入第一个会调多少个参数，直接调用就行了。

```
a = average(1,2,3,4,5)
```