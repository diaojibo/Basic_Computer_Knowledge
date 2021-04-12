## 冒号操作符和点号操作符

如下示例代码

```lua
local str = "abcde"
print("case 1:", str:sub(1, 2))
print("case 2:", str.sub(str, 1, 2))
```

冒号操作会带入一个 `self` 参数，用来代表 `自己`。而点号操作，只是 `内容` 的展开。

在函数定义时，使用冒号将默认接收一个 `self` 参数，而使用点号则需要显式传入 `self` 参数。

好比：

```lua
obj = { x = 20 }

function obj:fun1()
    print(self.x)
end
```

等价于

```lua
obj = { x = 20 }

function obj.fun1(self)
    print(self.x)
end
```

