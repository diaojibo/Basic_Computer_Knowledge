## LuaRocks详解

Luarocks是一个Lua包管理器，基于Lua语言开发，提供一个命令行的方式来管理Lua包依赖、安装第三方Lua包等，社区比较流行的包管理器之一，另还有一个LuaDist，Luarocks的包数量比LuaDist多

在做一些openresty的项目的时候，经常会借助一些第三方包来协助开发，为了方便管理，我们可以使用openresy官方的opm，或者lua的包管理工具[luarocks](https://luarocks.org/)，只不过opm的包数量还不是太多，用的较多的还是luarocks，现在只能期待opm社区不断的发展壮大了。

### Luarocks安装

根据官方的指引

```
$ wget https://luarocks.org/releases/luarocks-3.5.0.tar.gz
$ tar zxpf luarocks-3.5.0.tar.gz
$ cd luarocks-3.5.0
$ ./configure && make && sudo make install
$ sudo luarocks install luasocket
$ lua
Lua 5.3.5 Copyright (C) 1994-2018 Lua.org, PUC-Rio
> require "socket"
```

中途可能会遇到找不到头文件的问题

那干脆把Lua安装最新的算了

```
curl -R -O http://www.lua.org/ftp/lua-5.4.3.tar.gz
tar zxf lua-5.4.3.tar.gz
cd lua-5.4.3
make all test
```

注意，这只是装了Lua最新版，但是lua会默认安装在/usr/local/bin

所以我们需要设置环境变量，让系统优先感知到Lua最新版

比如说改bash_profile

```
PATH=/usr/local/bin:$PATH
```

然后重新执行一下bash profile就好

```
. ~/.bash_profile
```

安装lua之后，重新执行luarocks的make和make install就好

### rockspec编写

假如写好了Lua库想要发布到luarocks就得编写相应的rockspec文件

rockspec相当于一个lua库代码的元描述文件，实质上也是一个Lua文件

一个基本的rockspec模板如下

```lua
package = "LuaFruits"
version = "1.0-1"
source = {
   url = "..." -- We don't have one yet
}
description = {
   summary = "An example for the LuaRocks tutorial.",
   detailed = [[
      This is an example for the LuaRocks tutorial.
      Here we would put a detailed, typically
      paragraph-long description.
   ]],
   homepage = "http://...", -- We don't have one yet
   license = "MIT/X11" -- or whatever you like
}
dependencies = {
   "lua >= 5.1, < 5.4"
   -- If you depend on other rocks, add them here
}
build = {
   -- We'll start here.
}
```

在build这个选项的table中我们可以指示我们的包是如何被构建的,以下是最常用的builtin类型

```lua
build = {
   type = "builtin",
   modules = {

      -- A simple module written in Lua
      apricot = "src/apricot.lua",

      -- Note the required Lua syntax when listing submodules as keys
      ["apricot.seeds"] = "src/apricot/seeds.lua",

      -- A simple module written in C
      banana = "src/banana.c",

      -- C modules can span multiple files.
      cherry = {"src/cherry.c", "src/cherry_pie.c"},

      -- C modules also support an extended syntax, supporting
      -- cross-platform specifications of C defines, libraries and
      -- paths for external dependencies.
      date = {
         sources = {"src/date.c", "src/cali_date.c", "src/arab_date.c"},
         defines = {"MAX_DATES_PER_MEAL=50"},
         libraries = {"date"},
         incdirs = {"$(LIBDATE_INCDIR)"},
         libdirs = {"$(LIBDATE_LIBDIR)"}
      }
   }
}
```

Since modules written in Lua do not need to be compiled, LuaRocks only needs to know where they are in your source directory tree in order to copy them to the proper place on installation(see the "apricot" and "apricot.seeds" in the above example).

对于C代码，指定好路径，LuaRocks就会用合适的编译器去处理这个事情。