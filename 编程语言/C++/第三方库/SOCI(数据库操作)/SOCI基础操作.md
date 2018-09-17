## SOCI
SOCI是一个数据库操作的库，并不是ORM库，它仍旧需要用户编写sql语句来操作数据库，只是使用起来会更加方便

有以下几个特点:


1. 以stream方式输入sql语句
2. 通过into和use语法传递和解析参数
3. 支持连接池，线程安全

git项目地址 https://github.com/SOCI/soci

### 编译安装
我们按照下面操作从git上下载SOCI源码，然后编译。

编译的时候我们就按cmake的标准来走咯：
