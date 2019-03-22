## lua require路径问题
我们在lua中如果要引入一个模块：

``` lua
require("cjson")
```

这要求我们把相应的cjson.so或者cjson.lua放到lua脚本同级别的文件夹，找不到的化，lua会索引一大堆路径，去找这个库。

假如我们相用相对路径获取这个库，那就指定cpath指向这个so就好：

``` lua
package.cpath = "./cfg/cjson.so"
```