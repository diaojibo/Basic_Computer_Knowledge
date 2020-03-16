## CMake
你或许听过好几种 Make 工具，例如 GNU Make(**可见linux下的make笔记**) ，QT 的 qmake ，微软的 MS nmake，BSD Make（pmake），Makepp，等等。这些 Make 工具遵循着不同的规范和标准，所执行的 Makefile 格式也千差万别。这样就带来了一个严峻的问题：如果软件想跨平台，必须要保证能够在不同平台编译。而如果使用上面的 Make 工具，就得为每一种标准写一次 Makefile ，这将是一件让人抓狂的工作。

**CMake就是针对上面问题所设计的工具**：它首先允许开发者编写一种平台无关的 **CMakeList.txt** 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 工程。从而做到“Write once, run everywhere”。显然，CMake 是一个比上述几种 make 更高级的编译配置工具。

CMake 拥有比直接编写 Makefile 更直观的语法，更易于编写和维护。CLion也依赖于CMake。


具体用法可以参照网上手册，这里介绍简单用法。

### CMakeList
通过 CMake 管理的项目主目录下会有一个 CMakeLists.txt ，其中会包括工程包含哪些子目录等内容。 而在每个子目录下，也会包含一个 CMakeLists.txt， 用来管理该子目录中相关内容的构建。

```
cmake_minimum_required(VERSION 3.1)  # CMake 版本要求
PROJECT(hello)                       # 项目名称

aux_source_directory(. PROGRAM_SOURCE)  # 将当前目录所有文件添加到变量 PROGRAM_SOURCE 中

add_executable(hello ${PROGRAM_SOURCE}) # 指定目标可执行文件 hello 的源代码文件为 PROGRAM_SOURCE

add_library(hello  ${PROGRAM_SOURCE}) # 打包成静态库(.a)方便其他地方调用

add_library(hello  SHARED ${PROGRAM_SOURCE}) # 打包成动态库(.so)方便其他地方调用

```

以上示例，**add_executable就是最后会生成一个hello的可执行文件，而这个可执行文件的源文件是后者**

#### 添加外部头文件查找目录
当我们用到外部的库的时候，我们便需要添加外部库的头文件所在目录作为头文件查找目录。在 CMakeLists 中添加以下代码即可

```
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../
    ${CMAKE_CURRENT_SOURCE_DIR}../include)
```

CMake 中通过空格或者换行区分多个变量，上面的示例便是添加了两个目录到头文件查找路径中。

#### 添加外部链接库
通过以下代码可以添加外部链接库查找目录, 其中 CMAKE_CURRENT_SOURCE_DIR 是**内置宏**， 表示当前 CMakeLists.txt 所在目录：

```
link_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/../../lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../lib)
```

如果之前已经设定好了link_directiories,那么target_link_libraries根本就不需要设定相对路径了，直接链接就行了,动态库静态库都可以。

```
link_libraries(hiredis)
```

注意，如果只写库名，那默认优先链接动态库so，后.a，所以要链接静态库要先手动设置。

```
link_libraries(hiredis.a)
```

#### Set命令添加变量

可以用set命令添加变量

```
set(COMMON_OBJ A B C)

add_executable(Main ${COMMON_OBJ})
```

### cmake命令使用
两种使用方式：

 - cmake [option] <path-to-source\> 指向含有顶级CMakeLists.txt的那个目录
 - cmake [option] <path-to-existing-build\> 指向含有CMakeCache.txt的那个目录

第一种方式用于第一次生成cmake makefile，此后可以在build dir里直接cmake . **注意"."表示当前目录**,意思便是，我们假定在一个build文件夹，然后用cmake 命令指定cmakeList所在的位置，然后这里就会把一些预编译信息存到build文件夹里。cpp的使用习惯都是，经常需要开启一个build文件夹来存放编译的结果。如果当前目录已经有cmakeList，那就会使用第二种方式。

**Cmake推荐建立build目录进行编译，所有的中间文件都会生成在build目录下，需要删除时直接清空该目录即可。这就是所谓的外部编译方式。**

#### 常用选项
1. -G <generator-name\> 指定 **makefile生成器** 的名字。例如：cmake -G "MinGW Makefiles";注意generator是大小写敏感的，即使是在windows下。generator所用的命令(gcc,cl等)最好已经设置在环境变量PATH中。有个例外就是生成visual studio的工程不必设置环境变量，只要安装了对应的vs，cmake可以自动找到。

2. -D<var\>:<type\>=<value\> 添加变量及值到CMakeCache.txt中。注意-D后面不能有空格，type为string时可省略。例如：cmake -DCMAKE_BUILD_TYPE:STRING=Debug。MinGW Generator默认生成CMAKE_BUILD_TYPE为空，即release；NMake Generator默认生成CMAKE_BUILD_TYPE为Debug。

3. -U<globbing_expr\> 删除CMakeCache.txt中的变量。注意-U后面不能有空格,支持globbing表达式，比如*,?等。例如：cmake -UCMAKE_BUILD_TYPE。

#### 标准例子

下面是一个编译的标准例子：

```
git clone https://github.com/SOCI/soci.git
cd soci
mkdir build
cd build
cmake .. -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=/third_party/soci

make install

```

编译cmake时如果要指定编译器位置，那就要控制这个变量拉

```
CC=/usr/local/gcc9/bin/gcc CXX=/usr/local/gcc9/bin/g++ cmake .. -DCMAKE_INSTALL_PREFIX=/home/rockctli/third_bin 

```

**如果要指定libstdc++或者其他动态库的位置，那就**

```
cmake .. -DCMAKE_INSTALL_PREFIX=/home/rockctli/third_bin -DCMAKE_EXE_LINKER_FLAGS='-Wl,-rpath,/usr/local/gcc9/lib64'
```