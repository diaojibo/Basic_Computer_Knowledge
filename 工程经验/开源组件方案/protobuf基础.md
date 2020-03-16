## protobuf基础
protobuf（Protocol Buffers ）是Google的开源项目，是Google的中立于语言、平台，可扩展的用于序列化结构化数据的解决方案。其效率一般是XML XMPP的20倍以上，JSON的10倍以上

简单的说，protobuf是用来对数据进行序列化和反序列化。

相比较而言，Protobuf有如下优点：

 - 足够简单
 - 序列化后体积很小:消息大小只需要XML的1/10 ~ 1/3
 - 解析速度快:解析速度比XML快20 ~ 100倍
 - 多语言支持
 - 更好的兼容性,Protobuf设计的一个原则就是要能够很好的支持向下或向上兼容

### proto协议
可以采用如下的方式来定义消息类型的.proto文件

``` protobuf
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3;
}
```

以上定义的message的格式非常简单--每一个message类型都有一个或多个带有唯一编号的字段。等号后面的赋值就是所谓定义的**唯一编号**

#### 标识号
在消息定义中，每个字段都有唯一的一个数字标识符。这些标识符是用来在消息的二进制格式中识别各个字段的，一旦开始使用就不能够再改变。注：`[1,15]`之内的标识号在编码的时候会占用一个字节。`[16,2047]`之内的标识号则占用2个字节。所以应该为那些频繁出现的消息元素保留 `[1,15]`之内的标识号。切记：要为将来有可能添加的、频繁出现的标识号预留一些标识号。

最小的标识号可以从1开始，最大到2^29 - 1, or 536,870,911。不可以使用其中的`[19000－19999]`的标识号， Protobuf协议实现中对这些进行了预留。如果非要在.proto文件中使用这些预留标识号，编译时就会报警。

#### 字段规则
所指定的消息字段修饰符必须是如下之一：

 - required：一个格式良好的消息一定要含有1个这种字段。表示该值是必须要设置的；
 - optional：消息格式中该字段可以有0个或1个值（不超过1个）。
 - repeated：在一个格式良好的消息中，这种字段可以重复任意多次（包括0次）。重复的值的顺序会被保留。表示该值可以重复，相当于java中的List。


由于一些历史原因，基本数值类型的repeated的字段并没有被尽可能地高效编码。在新的代码中，用户应该使用特殊选项`[packed=true]`来保证更高效的编码。

``` protobuf
repeated int32 samples = 4 [packed=true];
```

*required是永久性的：在将一个字段标识为required的时候，应该特别小心。如果在某些情况下不想写入或者发送一个required的字段，将原始该字段修饰符更改为optional可能会遇到问题——旧版本的使用者会认为不含该字段的消息是不完整的，从而可能会无目的的拒绝解析。在这种情况下，你应该考虑编写特别针对于应用程序的、自定义的消息校验函数。Google的一些工程师得出了一个结论：使用required弊多于利；他们更 愿意使用optional和repeated而不是required。当然，这个观点并不具有普遍性。*

当用protocolbuffer编译器来运行.proto文件时，编译器将生成所选择语言的代码，这些代码可以操作在.proto文件中定义的消息类型，包括获取、设置字段值，将消息序列化到一个输出流中，以及从一个输入流中解析消息。



### Proto3

Protobuf公布出来也得到了大家的广泛的关注， 逐步地也得到了大家的认可，很多项目也采用Protobuf进行消息的通讯，还有基于Protobuf的微服务框架GRPC。在使用的过程中，大家也提出了很多的意见和建议，Protobuf也在演化，于2016年推出了Proto3。 Proto3简化了proto2的开发，提高了开发的效能，但是也带来了版本不兼容的问题。

目前Protobuf的稳定版本是3.9.2，于2019年9月23日发布。由于很多公司很早的就采用了Protobuf，所以很多项目还在使用proto2协议，目前是proto2和proto3同时在使用的状态。



官方的发布日志中列举了proto3的[改变](https://github.com/protocolbuffers/protobuf/releases/tag/v3.0.0)：

- 移除了原始值字段的出现逻辑。
- 移除了`required`字段
- 移除了缺省值
- 移除了`unknown`字段 （3.5中又加上了）
- 移除了扩展，使用`Any`代替
- 修复了未知的枚举值的语义
- 添加了map类型
- 添加了一些标准类似，比如time、动态数据的呈现
- 可以使用JSON编码代替二进制proto编码



简单例子：



```protobuf
syntax = "proto3";
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

它定义了一个message类型: `SearchRequest`， 它包含三个字段`query`、`page_number`、`result_per_page`,它会被protoc编译成不同的编程语言的相应对象，比如Java中的class、Go中的struct等。

字段是以`[ "repeated" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"`格式定义的。这个例子是一个简单的例子，采用了`type fieldName "=" fieldNumber`格式定义的。

比如第一个字段`query`, 首先是它的类型`string`，其次是字段的名称，然后是等号`=`, 之后是字段的序号，然后是分号。