## tarscpp使用
tarscpp的使用上跟tars node是略有不同的。

要使用tarscpp，我们首先需要在我们的机器上，编译好tars的使用环境。具体可以看官方文档，当具体的环境，一切配置都编译好之后，我们就可以开始后续开发(提供开发的tx编译机上一般已经配置就绪)

### 1.运行创建服务脚本
跟tars-node一样，我们最终目的是要发布一个tgz包到运营平台上。那么现在我们第一步，是要用一个脚本，创建出一个工程出来。

我们这里要先决定好App名,Server名，还有Servant的名字。此时路由名称当然也就得以确定下来了。

我们执行相应的脚本，比如我们决定开发的Obj路由名是 TestApp.HelloServer.Hello，在tx编译机下：

```
/usr/local/tars/cpp/script/create_tars_server.sh [App] [Server] [Servant]
```

命令执行后，会在当前目录下产生一系列工程文件 在TestApp/HelloServer下，可以看到

```
HelloServer.h HelloServer.cpp Hello.tars HelloImp.h HelloImp.cpp makefile
```

这个HelloServer.h 和helloServer.cpp,是脚本默认给你生成的工程入口文件，也就是说，main函数就是在这个xxxServer.cpp这里，当然你也完全可以自己手动再创建一个别的。这些Server文件，也被叫做 **服务的实现类**

除此之外，脚本还默认帮我们创建了一个初始tars文件，Hello.tars，这个tars文件没啥用，直接删了，然后我们自己写就好。但是仍旧要注意，我们的tars文件一般来说，都还是要以Servant命名，就好比刚才我们路由名称里，规定了Servant叫Hello，那我们自己写的tars文件也叫Hello.tars.

```
module TestApp
{

interface Hello
{
    int test();
};

};

```

### 2.tars2cpp生成tars的头文件

我们自己写好tars文件之后，名字是对应的Servant名字。接下来就要使用tars2cpp工具，这在tx编译机的/usr/local/tars/cpp/tools/tars2cpp上

我们使用命令

```
/usr/local/tars/cpp/tools/tars2cpp hello.tars
```

会生成一个hello.h文件，这个文件就是我们tars接口转成c++对应的封装了。

有了这个头文件，我们接下来就应该在HelloImp里实现那些我们写在tars文件当中的接口

### 3.实现Imp文件，完成接口
我们的初始脚本，是帮我们生成了一个HelloImp文件的，里面的内容是：

HelloImp.h

``` c++
#ifndef _HelloImp_H_
#define _HelloImp_H_

#include "servant/Application.h"
#include "Hello.h"

/**
 * HelloImp继承hello.h中定义的Hello对象
 *
 */
class HelloImp : public TestApp::Hello
{
public:
    /**
     *
     */
    virtual ~HelloImp() {}

    /**
     * 初始化，Hello的虚拟函数，HelloImp初始化时调用
     */
    virtual void initialize();

    /**
     * 析构，Hello的虚拟函数，服务析构HelloImp退出时调用
     */
    virtual void destroy();


};
/////////////////////////////////////////////////////
#endif
```

hello.cpp

``` c++
#include "HelloImp.h"
#include "servant/Application.h"

using namespace std;

//////////////////////////////////////////////////////
void HelloImp::initialize()
{
    //initialize servant here:
    //...
}

//////////////////////////////////////////////////////
void HelloImp::destroy()
{
    //destroy servant here:
    //...
}
```


我们这时，要在imp的头文件或者cpp文件里，加上test的实现,比如我们在h头文件里，加上test函数的实现。

**千万也不要忘了，把我们上一步的头文件，在实现类的时候继承进去，同时也要引用。因为默认我们生成的文件里已经引入了Servant.h,所以如果我们上一步tars文件转成h文件的名字不是Servant对应，那就要改成引入并继承相应的头文件**

``` c++
#ifndef _HelloImp_H_
#define _HelloImp_H_

#include "servant/Application.h"

//引入tars文件生成的头文件
#include "Hello.h"

/**
 * HelloImp继承hello.h中定义的Hello对象
 * 这里一定要注意，这一步的hello.h是由上一步tars2cpp生成的
 *
 */
class HelloImp : public TestApp::Hello
{
public:
    /**
     *
     */
    virtual ~HelloImp() {}

    /**
     * 初始化，Hello的虚拟函数，HelloImp初始化时调用
     */
    virtual void initialize();

    /**
     * 析构，Hello的虚拟函数，服务析构HelloImp退出时调用
     */
    virtual void destroy();

    /**
     * 实现tars文件中定义的test接口
     */
    virtual int test(tars::TarsCurrentPtr current) { return 0;};

};
/////////////////////////////////////////////////////
#endif
```

### 4.确认Server文件

在HelloServer.cpp入口文件中，addServant中写的Obj路由名，要跟发布网页上的对应，所以记得修改。

``` c++
addServant<HelloTestImp>(ServerConfig::Application + "." + ServerConfig::ServerName + ".HelloTestObj");
```

### 5.编写客户端
客户端调用只要包含几个特定头文件就可以了，一个是Servant对应的头文件，可由tars文件生成。

然后include "servant/Communicator.h"


``` c++
int main(int argc,char ** argv)
{
    Communicator comm;
    try
    {
        HelloPrx prx;
        comm.stringToProxy("TestApp.HelloServer.HelloObj@tcp -h 10.120.129.226 -p 20001" , prx);
        try
        {
            string sReq("hello world");
            string sRsp("");

            int iRet = prx->testHello(sReq, sRsp);

            cout<<"iRet:"<<iRet<<" sReq:"<<sReq<<" sRsp:"<<sRsp<<endl;
        }

        catch(exception &ex)
        {
            cerr << "ex:" << ex.what() << endl;
        }

        catch(...)
        {
            cerr << "unknown exception." << endl;
        }
    }
    catch(exception& e)
    {
        cerr << "exception:" << e.what() << endl;
    }
    catch (...)
    {
        cerr << "unknown exception." << endl;
    }
    return 0;
}
```


### 编译
在工程代码目录下，输入命令：

```
make cleanall
make
make tar
```

执行完这三个命令之后，tgz文件便生成了，然后就可以跟tars-node一样的步骤，上传到管理平台。
