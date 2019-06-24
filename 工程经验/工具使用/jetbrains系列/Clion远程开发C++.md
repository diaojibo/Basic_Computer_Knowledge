## Clion远程开发c++
windows上使用clion远程开发linux c++程序。

linux上当然先要预装这两个程序：

 - Cmake
 - gdb

怎么安装上面两个，不赘述。

然后在settings的toolchain里设置：

![](image/clion6.png)

![](image/clion7.png)

![](image/clion8.png)

这种方法相当于利用了远程的编译器，然后调试。