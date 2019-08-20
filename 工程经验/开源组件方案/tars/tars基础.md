## tars基础
Tars是tencent开发的高性能RPC开发框架，配套一体化的运营管理平台，并通过伸缩调度，实现运维半托管服务。

使用Tars框架，可以让 **开发更简单，聚焦业务逻辑，让运营更高效，一切尽在掌握**。

RPC开发框架解决的无非就是机器节点之间的进程远程调用的问题。客户端进行函数调用，服务端进程执行最后返回结果。

![](image/rpc0.jpg)

要使用Tars,我们还是需要理解其中的一些概念。

### Tars语言
Tars语言是一种类c++标识符的语言，**用于生成具体的服务接口文件**

为了使各个节点的进程之间互相通信，我们用Tars语言来约定一些接口文件。然后服务端和客户端就可以根据这个Tars接口文件来进行相应的开发。

如下图就是一个Tars文件的示例：

``` Tars
// NodeJsComm.tars
module TRom
{
    struct User_t
    {
        0 optional int id = 0;
        1 optional int score = 0;
        2 optional string name = "";
    };

    struct Result_t
    {
        0 optional int id = 0;
        1 optional int iLevel = 0;
    };

    interface NodeJsComm
    {
        int test();

        int getall(User_t stUser, out Result_t stResult);

        int getUsrName(string sUsrName, out string sValue1, out string sValue2);

        int secRequest(vector<byte> binRequest, out vector<byte> binResponse);
    };
};
```

如上图，是一个名为NodeJsComm的tars文件，上面约定了两种类型(Struct结构体),User_t和Result_t. 然后约定了一个接口，这一个接口封装了一系列函数调用，如test,getall等。

根据这个Tars接口定义文件，Server端就可以开发NodeJsComm这个接口了，实现test，getUsrName等函数实现。客户端根据这个Tars文件，就可知道这个接口有哪些函数可以调用，并且如何封装一些参数(比如参数是定义的结构体)。

关于Tars语言更详细的定义，在[Tars TUP](https://github.com/Tencent/Tars/blob/master/docs/tars_tup.md)说明文档里会说到。

### Tars协议
刚才说了Tars语言可以用来编写Tars接口文件。那么Tars协议，则规定了Tars二进制数据流在网络中如何传输和解析。

根据Tars框架来进行远程调用过的时候，网络中的Tars数据包是这样构成的：

```
| 头信息 | 实际数据 |
```

分为header信息和data实际数据信息。

其中header包的内容为：

```
| Type(4 bits) | Tag 1(4 bits) | Tag 2(1 byte) |
```

Tag 2是可选的，当Tag的值不超过14时，只需要用Tag 1就可以表示；当Tag的值超过14而小于256时，Tag 1固定为15，而用Tag 2表示Tag的值。Tag不允许大于255。

Type表示类型，用4个二进制位表示，取值范围是0~15，用来标识该数据的类型。不同类型的数据，其后紧跟着的实际数据的长度和格式都是不一样的，详见一下的[类型表](https://github.com/Tencent/Tars/blob/master/docs/tars_tup.md)。

比如说有如下Tars接口文件定义了某struct

```
struct TestInfo
{
    1  require  int    ii  = 34;
    2  optional string s   = "abc";
};

struct TestInfo2
{
    1  require TestInfo  t;
    2  require int       a = 12345;
};

```

那么TestInfo2在网络中传输时，编码则为下图：

![](image/tars0.png)

#### 底层消息封装
客户端和服务端之间的通信请求和响应，也是经过了Tars的封装。以下为请求包和相应包的Tars结构：

```
//请求包体
struct RequestPacket
{
    1  require short        iVersion;         //版本号
    2  optional byte        cPacketType;      //包类型
    3  optional int         iMessageType;     //消息类型
    4  require int          iRequestId;       //请求ID
    5  require string       sServantName;     //servant名字
    6  require string       sFuncName;        //函数名称
    7  require vector<byte> sBuffer;          //二进制buffer
    8  optional int         iTimeout;         //超时时间（毫秒）
    9  optional map<string, string> context;  //业务上下文
    10 optional map<string, string> status;   //框架协议上下文
};
```

```
//响应包体
struct ResponsePacket
{
    1 require short         iVersion;       //版本号
    2 optional byte         cPacketType;    //包类型
    3 require int           iRequestId;     //请求ID
    4 optional int          iMessageType;   //消息类型
    5 optional int          iRet;           //返回值
    6 require vector<byte>  sBuffer;        //二进制流
    7 optional map<string, string> status;  //协议上下文
    8 optional string       sResultDesc;    //结果描述
};

//返回值
const int TAFSERVERSUCCESS       = 0;       //服务器端处理成功
const int TAFSERVERDECODEERR     = -1;      //服务器端解码异常
const int TAFSERVERENCODEERR     = -2;      //服务器端编码异常
const int TAFSERVERNOFUNCERR     = -3;      //服务器端没有该函数
const int TAFSERVERNOSERVANTERR  = -4;      //服务器端没有该Servant对象
const int TAFSERVERRESETGRID     = -5;      //服务器端灰度状态不一致
const int TAFSERVERQUEUETIMEOUT  = -6;      //服务器队列超过限制
const int TAFASYNCCALLTIMEOUT    = -7;      //异步调用超时
const int TAFINVOKETIMEOUT       = -7;      //调用超时
const int TAFPROXYCONNECTERR     = -8;      //proxy链接异常
const int TAFSERVEROVERLOAD      = -9;      //服务器端超负载,超过队列长度
const int TAFADAPTERNULL         = -10;     //客户端选路为空，服务不存在或者所有服务down掉了
const int TAFINVOKEBYINVALIDESET = -11;     //客户端按set规则调用非法
const int TAFCLIENTDECODEERR     = -12;     //客户端解码异常
const int TAFSERVERUNKNOWNERR    = -99;     //服务器端位置异常
```

### TUP
TUP（Tars Uni-Protocol的简称），Tars统一协议，是基于Tars编码的命令字（Command）层协议的封装。

简而言之，基于Tars 统一协议，就可以把Tars文件翻译成多语言的对应的C/S接口文件来方便多语言使用。

写好Tars文件后，我们需要用到各种工具来把Tars文件转化为各语言的接口文件，比如使用tars2cpp,tars2java等。

所以当我们基于Tars框架进行开发的时候,首先就是要服务端，客户端约定好Tar接口文件，然后由服务端定义好这个Tars文件。假如前后端都是使用不同的语言，拿到tars文件后就可以用tars2xxx工具生成对应的接口文件。服务端实现这个文件，客户端则可以根据这个文件的进行接口调用。

### 参考
[tars github](https://github.com/Tencent/Tars/blob/master/README.zh.md)
