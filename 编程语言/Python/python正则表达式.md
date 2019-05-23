## 正则表达式
python作为最常用的脚本语言，必然少不了用来处理文本，写正则。

这里记一些快速的用法：

首先要`import re`

然后调用re的的方法

### 贪婪与不贪婪
在开始用正则表达式之前，还得先说下贪婪与不贪婪的问题。

正则表达式通常用于在文本中查找匹配的字符串。Python里数量词**默认是贪婪的**，总是尝试匹配尽量多的字符，记准这一个原则，才能使我们查找的时候不出错。

例如：正则表达式"ab*"如果用于查找"abbbc"，将找到"abbb"。而如果使用非贪婪的数量词"ab*?"，将找到"a"。

ps：匹配标签的时候，我们尤其需要注意这一点，中间大段内容用星号时要指定?符号，这样才会把每一块提取出来。


### 常用符号

 - 最常用的就是. 匹配任意字符，但是，这**不包括换行符**，注意了，如果要包括换行符，那就用`[\S\s]`
 - \S 非空白字符
 - \s 空白字符
 - [...] 中括号内表示可以匹配其中的任意字符

### Pattern
跟很多语言一样，推荐使用时先把正则表达式编译成一个pattern对象。

``` python
# encoding: UTF-8 
import re 
 
# 将正则表达式编译成Pattern对象 
pattern = re.compile(r'world') 
```

patern对象则有许多有用的函数

#### findall
提取所有匹配到的正则模式块：

``` python
import re
 
p = re.compile(r'\d+')
print p.findall('one1two2three3four4')
 
### output ###
# ['1', '2', '3', '4']
```

findall还可以在正则匹配里进行分组抽取，比如用括号输入在re.compile当中，那findall出来就是一个列表，每个列表里都是一个元组，元祖里就是抽取出来的一个个小分组。

#### search方法
拿字符串去匹配搜索所有匹配的正则，匹配成功会返回一个match对象。match对象后面会有讲解。

``` python
# encoding: UTF-8 
import re 
 
# 将正则表达式编译成Pattern对象 
pattern = re.compile(r'world') 
 
# 使用search()查找匹配的子串，不存在能匹配的子串时将返回None 
# 这个例子中使用match()无法成功匹配 
match = pattern.search('hello world!') 
 
if match: 
    # 使用Match获得分组信息 
    print match.group() 
 
### 输出 ### 
# world
```

search用于查找字符串中所有符合的模式，跟match方法用法不同，不要搞混。

#### sub方法
替换string中每一个匹配的子串后返回替换后的字符串。

sub方法的使用方法要明确：

1. 首先要定义要替换掉的串模式
2. 用替换串去替换

```
#!python
>>> p = re.compile( '(blue|white|red)')
>>> p.sub( 'colour', 'blue socks and red shoes')
'colour socks and colour shoes'
>>> p.sub( 'colour', 'blue socks and red shoes', count=1)
'colour socks and red shoes'
```

上面先定义了一个模式，也就是啥串需要换。然后直接sub把串换掉。


### Match对象
Match对象是一次匹配的结果，包含了很多关于此次匹配的信息，可以使用Match提供的可读属性或方法来获取这些信息。

最终要的方法是：

group([group1, …]): 
获得一个或多个分组截获的字符串；指定多个参数时将以元组形式返回。group可以使用编号也可以使用别名；编号0代表整个匹配的子串；**不填写参数时，返回group(0)；没有截获字符串的组返回None；截获了多次的组返回最后一次截获的子串**。