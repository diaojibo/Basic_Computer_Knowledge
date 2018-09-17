## Cmake常用变量
在Cmake使用基础里，已经提到过可以通过SET来设置变量。

```
SET(HELLO_SRC main.c SOURCE_PATH)
```

如上图，设置了HELLO_SRC这个变量，这个变量内容为main.c这个文件和SOURCE_PATH变量下的文件。

### 常用变量

#### 1. CMAKE_BINARY_DIR/PROJECT_BINARY_DIR /\_BINARY_DIR
这三个变量指代的 **内容是一致的**，如果是in source 编译，指得就是 **工程顶层目录**，如果是out-of-source 编译，指的是工程编译发生的目录。PROJECT_BINARY_DIR 跟其他指令稍有区别，现在，你可以理解为他们是一致的。

#### 2.CMAKE_SOURCE_DIR/PROJECT_SOURCE_DIR /\_SOURCE_DIR

这三个变量指代的内容是一致的，不论采用何种编译方式，都是工程顶层目录。

也就是在in source 编译时，他跟CMAKE_BINARY_DIR 等变量一致。

#### 3.CMAKE_CURRENT_SOURCE_DIR
指的是当前处理的CMakeLists.txt 所在的路径，比如上面我们提到的src子目录(有些工程CMakeLists就放在src目录下)。

#### 4.CMAKE_CURRRENT_BINARY_DIR
如果是in-source 编译，它跟CMAKE_CURRENT_SOURCE_DIR 一致，如果是out-of-source 编译，他指的是target 编译目录。使用我们上面提到的ADD_SUBDIRECTORY(src bin) 可以更改这个变量的值。使用SET(EXECUTABLE_OUTPUT_PATH < 新路径>)并不会对这个变量造成影响，它仅仅修改了最终目标文件存放的路径。

#### 5.CMAKE_CURRENT_LIST_FILE
输出调用这个变量的CMakeLists.txt 的完整路径

#### 6.CMAKE_CURRENT_LIST_LINE
输出这个变量所在的行

#### 7.CMAKE_MODULE_PATH
这个变量用来定义自己的cmake 模块所在的路径。如果你的工程比较复杂，有可能会自己编写一些cmake 模块，这些cmake 模块是随你的工程发布的，为了让cmake 在处理CMakeLists.txt 时找到这些模块，你需要通过SET指令，将自己的cmake 模块路径设置一下。

比如

SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake) 这时候你就可以通过INCLUDE 指令来调用自己的模块了。


#### 8.CMAKE_INSTALL_PREFIX(重)
这个变量要单独拿出来说一下，因为安装第三方软件的时候经常有需要用到它。我们在使用cmake命令的时候，经常会用-D指令来添加这个参数。这个参数其实就是为我们指定之后项目编译到的目录，比如：

```
cmake .. -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/opt/third_party/soci
```

上面cmake命令执行之后，然后make install，就安装了一个soci第三方库，把它指定安装到了`/opt/third_party/soci`这个目录。它相当于，通过影响cmake的内置变量，从而生成的makefile中内置的默认安装目录也不一样了，所以最后执行make install，直接就把它安装到对应位置去了。

当然我们其实也可以这么做：

```
cmake ..
make
make install DESTDIR=/opt/third_party/soci
```

DESTDIR只在make install时起作用，且和Makefile是由什么工具生成的没有关系(是GNU make的默认规定)，也能帮助我们指定安装程序的目录位置。

ps：除此之外，打包时常见的做法当然还有configure --prefix，利用configure脚本来进行配置。

### 参考
[cmake实践](https://www.kancloud.cn/itfanr/cmake-practice/82989)
