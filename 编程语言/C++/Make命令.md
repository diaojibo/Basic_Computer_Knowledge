## Make与编译
代码变成可执行文件，叫做编译（compile）；先编译这个，还是先编译那个（即编译的安排），叫做构建（build）。

Make是最常用的构建工具，诞生于1977年，主要用于C语言的项目。但是实际上 ，任何只要某个文件有变化，就要重新构建的项目，都可以用Make构建。

**make命令需要一定的规则去进行编译和构建，这个规则就是Makefile文件**。

总之，make只是一个根据指定的Shell命令进行构建的工具。它的规则很简单，你规定要构建哪个文件、它依赖哪些源文件，当那些文件有变动时，如何重新构建它。

### Makefile文件
构建规则都写在Makefile文件里面，要学会如何Make命令，就必须学会如何编写Makefile文件。

Makefile文件由一系列规则（rules）构成。每条规则的形式如下：

```
<target> : <prerequisites>
[tab]  <commands>
```

上面第一行冒号前面的部分，叫做"目标"（target），冒号后面的部分叫做"前置条件"（prerequisites）；第二行必须由一个tab键起首，后面跟着"命令"（commands）。

"目标"是必需的，不可省略；"前置条件"和"命令"都是可选的，但是两者之中必须至少存在一个。

**前置条件指的是，编译这个target，有什么东西是必须的。**

每条规则就明确两件事：构建目标的前置条件是什么，以及如何构建。下面就详细讲解，每条规则的这三个组成部分。

#### 目标target
一个目标（target）就构成一条规则。目标通常是文件名，指明Make命令所要构建的对象，比如 a.txt 。目标可以是一个文件名，也可以是多个文件名，之间用空格分隔。

除了文件名，目标还可以是某个操作的名字，这称为"伪目标"（phony target）。

```
clean:
      rm *.o
```

上面代码的目标是clean，它不是文件名，而是一个操作的名字，属于"伪目标 "，作用是删除对象文件。

```
make  clean
```

但是，如果当前目录中，正好有一个文件叫做clean，那么这个命令不会执行。因为Make发现clean文件已经存在，就认为没有必要重新构建了，就不会执行指定的rm命令。
为了避免这种情况，可以明确声明clean是"伪目标"，写法如下。

```
.PHONY: clean
clean:
        rm *.o temp
```

声明clean是"伪目标"之后，make就不会去检查是否存在一个叫做clean的文件，而是每次运行都执行对应的命令。像.PHONY这样的内置目标名还有不少，可以查看手册。

**如果Make命令运行时没有指定目标，默认会执行Makefile文件的第一个目标**。

#### 前置条件（prerequisites）
前置条件通常是一组文件名，之间用空格分隔。它指定了"目标"是否重新构建的判断标准：只要有一个前置文件不存在，或者有过更新（前置文件的last-modification时间戳比目标的时间戳新），"目标"就需要重新构建。

```
result.txt: source.txt
    cp source.txt result.txt
```

上面代码中，构建 result.txt 的前置条件是 source.txt 。如果当前目录中，source.txt 已经存在，那么make result.txt可以正常运行

#### 命令（commands）
命令（commands）表示如何更新目标文件，由一行或多行的Shell命令组成。它是构建"目标"的具体指令，它的运行结果通常就是生成目标文件。

每行命令之前必须有一个tab键。如果想用其他键，可以用内置变量.RECIPEPREFIX声明。


#### 回声（echoing）
正常情况下，make会打印每条命令，然后再执行，这就叫做回声（echoing）。

在命令的前面加上@，就可以关闭回声。

```
test:
    @# 这是测试
```

#### 模式匹配
Make命令允许对文件名，进行类似正则运算的匹配，主要用到的匹配符是%。比如，假定当前目录下有 f1.c 和 f2.c 两个源码文件，需要将它们编译为对应的对象文件。

```
%.o: %.c
```

上面命令，根据正则匹配

```
f1.o: f1.c
f2.o: f2.c
```

#### 定义变量
Makefile 允许使用等号自定义变量。

```
txt = Hello World
test:
    @echo $(txt)
```

调用时，变量需要放在 $( ) 之中。

#### 内置变量
Make命令提供一系列内置变量，比如，$(CC) 指向当前使用的编译器，$(MAKE) 指向当前使用的Make工具。这主要是为了跨平台的兼容性


### 范例Example

```
main : main.o a.o b.o
g++ -o main main.o a.o b.o

main.o : main.c main.h b.h
g++ -c main.c

a.o : a.c a.h
g++ -c a.c

b.o : b.c b.h
g++ -c b.c

```

如图上面是一个简易的makefile文件，用make或者make main既能自动生成可执行文件main。

在执行main命令是，依赖于main.o a.o b.o三个目标文件(库文件)，因为需要把这三个编译好的库文件链接在一起才能生成一个可执行文件。

由于没有三个目标文件，make程序会去下面找编译出这三个目标文件的方法，找到源文件main.c a.h b.h 等，然后编译出这些目标文件之后会再回到main处将他们链接起来。

### Make命令参数
我们用gcc编译程序时，可能会用到“-I”（大写i），“-L”（大写l），“-l”（小写l）等参数

例如：

```
gcc -o hello hello.c -I /home/hello/include -L /home/hello/lib -lworld
```

上面这句表示在编译hello.c时：

 - -I /home/hello/include表示将/home/hello/include目录作为第一个寻找头文件的目录，寻找的顺序是：/home/hello/include-->/usr/include-->/usr/local/include

 - -L 指定库文件的目录，也就是如果链接时候，需要的动态库或是静态库不在默认的路径中，就需要用-Lxxx指定库文件路径。寻找的顺序是：/home/hello/lib-->/lib-->/usr/lib-->/usr/local/lib

 - -lworld表示在上面的lib的路径中寻找libworld.so动态库文件（如果gcc编译选项中加入了“-static”表示寻找libworld.a静态库文件）,也就是找.so动态库或者.a静态库。

 综上，-I指定头文件目录，-L指定库文件目录，-l指定库名。


所以我们在使用C++第三方库的时候，无非就是要做好两点，第一，用-I参数指定好对应的头文件存放处，头文件指示了有哪些函数可以调用。第二，用-L指定好库存放的地方，并且-l指定对应要加载进的库名。

### 编译第三方库
我们需要编译第三方库的步骤就是这么几个：

1. git clone项目下来
2. 看看是make项目还是cmake，如果是cmake，走cmake编译流程
3. make项目，直接尝试make，并且指定install的目录到本地指定项目即可。根据一般的规定，只要指定DESTDIR就好

```
make install DESTDIR=../install
```

更进一步，直接去指定make脚本里的相关变量：

比如

![](image/make0.png)

我们在makefile里看到安装路径，直接改这里：

```
make install INSTALL_INCLUDE_PATH=./install/include INSTALL_LIBRARY_PATH=./install/lib
```
