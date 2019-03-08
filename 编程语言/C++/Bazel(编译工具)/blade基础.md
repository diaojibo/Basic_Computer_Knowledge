## blade编译工具基础
说到bazel，其实还得说到Blade这个编译构建工具,Blade是腾讯开源的构建工具，实现也参考了Bazel，这里也记录一些使用时的注意事项。


### 代码项目根目录
**Blade通过BLADE_ROOT文件来判定大型项目的根目录**，所以一个大型的C++项目，一定要合理组织源码，然后在根目录下放置一个`BLADE_ROOT`文件。

Blade获取源代码根的方法是，无论当前从哪一级子目录运行，都从当前目录开始向上查找BLADE_ROOT文件，有这个文件的目录即为源代码树的根

### BUILD文件
和bazel一样，在项目(子项目)的目录结构下，我们可以放置一个BUILD文件。代表这个目录要进行项目构建(编译)。

BUILD文件通过一组目标描述函数描述一个目标的源文件，所依赖的其他目标，以及其它一些属性等。BUILD 文件很简单，如common/base/string/BUILD：

```
cc_library(
    name = 'string',       # 目标名
    srcs = [
        'algorithm.cpp',
        'string_number.cpp',
        'string_piece.cpp',
        'format.cpp',
        'concat.cpp'
    ],                               # 源代码集合

    deps = ['//common/base:int'] # 直接依赖集合，没有可以不写
)
```

这些描述都是自解释的，只需列出目标名，源文件名和依赖名即可。在写完这个BUILD文件后，用户即可以通过命令行构建这个library目标。

写好配置文件以后，执行 `blade build` 命令即可执行编译。

除此之外还有一些属性定义很常见：

 - **export_incs**: Export_incs 表示对外可见的include文件查找路，依赖于此target的模块编译时将会自动将此目录加入到头文件查到队列中;依赖是传递的。
 - **prebuilt**: 有时候我们在网上使用一些第三方库，都是使用Makefile来进行构建的。这时候Blade就不太适用了，这时我们就要在生成的library加上prebuilt=True；使用这个参数表示不从源码构建。对应的二进制文件必须存在 lib{32,64}\_{release,debug} 这样的子目录中。**所以要将lib文件软链接到 lib\{32,64\}\_{release,debug}**.

### 利用环境变量指定不同的gcc编译
事实上，在编译代码的时候还可采用不同的gcc版本来进行编译。

我们要事先指定好几个环境变量：

```
TOOLCHAIN_DIR，默认为空
CPP，默认为cpp
CXX，默认为c++
CC，默认为gcc
LD，默认为c++
```

最简单的方式，就是我们把TOOLCHAIN_DIR这个变量直接指向我们相拥的gcc的bin目录位置就行了，然后再接blade build命令。

比如我们可以：

```
TOOLCHAIN_DIR=/usr/bin blade build
```

直接在执行命令之前就指定gcc目录啦，这就可以编译了。


### 第三方库工程的组织
经过我对公司项目的观察，发现第三方库整合进项目一般都如下组织。

第三方库一般都是使用makefile来进行编译的，所以我们不能直接用Blade模式构建，而是要预先构建好二进制文件，然后放到库文件夹的lib{32,64}\_{release,debug} 文件夹中。比如把.a静态二进制库文件放到lib64文件夹中。

接着我们再把第三方库的头文件,也就是.h和.hpp文件全部移到第三方目录下。最后在BUILD文件里加一个prebuilt=true，export_incs=./就可以了。接下来只要别人import你这个项目，第三方库就可用了。


## 参考
BLADE目前已经在GITHUB上更新维护：

[Typhoon-Blade github地址](https://github.com/chen3feng/typhoon-blade)