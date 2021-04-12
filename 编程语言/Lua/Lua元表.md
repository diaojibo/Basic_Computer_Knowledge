## 元表

在 Lua 5.1 语言中，元表 *(metatable)* 的表现行为类似于 C++ 语言中的操作符重载，例如我们可以重载 "__add" 元方法 *(metamethod)*，来计算两个 Lua 数组的并集；或者重载 "__index" 方法，来定义我们自己的 Hash 函数。Lua 提供了两个十分重要的用来处理元表的方法，如下：

- setmetatable(table, metatable)：此方法用于为一个表设置元表。
- getmetatable(table)：此方法用于获取表的元表对象。

```
local mytable = {}
local mymetatable = {}
setmetatable(mytable, mymetatable)
```

### 修改表的操作符行为

通过重载 "__add" 元方法来计算集合的并集实例：

```lua
local set1 = {10, 20, 30}   -- 集合
local set2 = {20, 40, 50}   -- 集合

-- 将用于重载__add的函数，注意第一个参数是self
local union = function (self, another)
    local set = {}
    local result = {}

    -- 利用数组来确保集合的互异性
    for i, j in pairs(self) do set[j] = true end
    for i, j in pairs(another) do set[j] = true end

    -- 加入结果集合
    for i, j in pairs(set) do table.insert(result, i) end
    return result
end
setmetatable(set1, {__add = union}) -- 重载 set1 表的 __add 元方法

local set3 = set1 + set2
for _, j in pairs(set3) do
    io.write(j.." ")               -->output：30 50 20 40 10
end
```

除了加法可以被重载之外，Lua 提供的所有操作符都可以被重载：

| 元方法     |                             含义                             |
| :--------- | :----------------------------------------------------------: |
| "__add"    |                            + 操作                            |
| "__sub"    |                - 操作 其行为类似于 "add" 操作                |
| "__mul"    |                * 操作 其行为类似于 "add" 操作                |
| "__div"    |                / 操作 其行为类似于 "add" 操作                |
| "__mod"    |                % 操作 其行为类似于 "add" 操作                |
| "__pow"    |             ^ （幂）操作 其行为类似于 "add" 操作             |
| "__unm"    |                         一元 - 操作                          |
| "__concat" |                    .. （字符串连接）操作                     |
| "__len"    |                            # 操作                            |
| "__eq"     | == 操作 函数 getcomphandler 定义了 Lua 怎样选择一个处理器来作比较操作 仅在两个对象类型相同且有对应操作相同的元方法时才起效 |
| "__lt"     |                            < 操作                            |
| "__le"     |                           <= 操作                            |

除了操作符之外，如下元方法也可以被重载，下面会依次解释使用方法：

| 元方法        |               含义                |
| :------------ | :-------------------------------: |
| "__index"     |   取下标操作用于访问 table[key]   |
| "__newindex"  | 赋值给指定下标 table[key] = value |
| "__tostring"  |           转换成字符串            |
| "__call"      |      当 Lua 调用一个值时调用      |
| "__mode"      |      用于弱表(*week table*)       |
| "__metatable" |     用于保护metatable不被访问     |