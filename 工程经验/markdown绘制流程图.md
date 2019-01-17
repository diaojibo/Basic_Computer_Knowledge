## markdown 绘制流程图
MarkDown中的流程图语法叫flow,这算是markdown的高级语法，在编写markdown flow代码时，我们示例如下，用` ``` flow ``` ` 来括起来。

``` flow
st=>start: 开始
rain?=>condition: 今天有雨吗？
takeAnUmbrella=>operation: 带伞
go=>operation: 出门
e=>end: 结束

st->rain?
rain?(yes)->takeAnUmbrella->go
rain?(no)->go->e
```

规则其实也不难理解，定义模块时，概括出来如下：

```
模块标识=>模块关键词(type): 模块名称(label)
```

模块标识可以理解为变量名，如上流程图的st，模块关键词相当于可以理解成这个模块的type，比如是个start的话，就是会画成一个圆形，是condition那肯定就是个菱形，是operation那就是一个矩形。`=>`符号则可以定义连接关系的起点终点。

我们仔细观察flow的代码，可以看到代码分为两块，一块是模块的定义，也就是st，rain?还有go这些变量的赋值操作，第二块是连接关系的定义，用`->`操作符来控制。同时，condition括号里可以填写yes和no作为关键词，表示分支，除了yes和no以外，还可以用left或者right来表示


### 关键词list
下面总结一下定义模块，连接关系时候要用到的关键词：

1. start,end, 表示程序的开始与结束
2. operation, 表示程序的处理块
3. subroutine, 表示子程序块
4. condition, 表示程序的条件判断
5. inputoutput, 表示程序的出入输出
6. right,left, 表示箭头在当前模块上的起点(默认箭头从下端开始)
7. yes,no, 表示condition判断的分支(其可以和right,left同时使用)
