## linux文本处理

### find文件查找
多个笔记里都提到了find可用于文件查找

查找txt和pdf文件:

```
find . \( -name "*.txt" -o -name "*.pdf" \) -print
```

正则方式查找.txt和pdf:

```
find . -regex  ".*\(\.txt|\.pdf\)$"
```

#### 按类型搜索

加入 -type参数

```
-type f 文件 / l 符号链接 / d 目录
```

列出所有目录

```
find . -type d -print  //只列出所有目录
```


#### 按时间搜索
 - -atime 访问时间 (单位是天，分钟单位则是-amin，以下类似）
 - -mtime 修改时间 （内容被修改）
 - -ctime 变化时间 （元数据或权限变化）


最近7天内被访问过的所有文件:

```
find . -atime 7 -type f -print
```

查询7天前被访问过的所有文件:

```
find . -atime +7 type f -print
```


#### 按大小搜索：
寻找大于2k的文件:

```
find . -type f -size +2k
```

#### 找到后的后续动作
删除用delete，**执行用exec**！

将找到的文件全都copy到另一个目录:

```
find . -type f -mtime +10 -name "*.txt" -exec cp {} OLD \;
```

#### grep文本搜索
常用参数

 - -o 只输出匹配的文本行 VS -v 只输出没有匹配的文本行
 - -c 统计文件中包含文本的次数
 - grep -c “text” filename
 - -n 打印匹配的行号
 - -i 搜索时忽略大小写
 - -l 只打印文件名
 - -r 递归的对目录下的所有文件（包括子目录）进行 grep


在多级目录中对文本递归搜索(程序员搜代码的最爱）:

```
grep "class" . -Rn
```

综合应用：将日志中的所有带where条件的sql查找查找出来:

```
cat LOG.* | tr a-z A-Z | grep "FROM " | grep "WHERE" > b
```

#### xargs 命令行参数转换
xargs命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。它擅长将标准输入数据转换成命令行参数，xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。

xargs也可以将单行或多行文本输入转换为其他格式，例如多行变单行，单行变多行。xargs的默认命令是echo，**空格是默认定界符**。这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将被空格取代。xargs是构建单行命令的重要组件之一。

定义一个测试文件，内有多行文本数据：

```
cat test.txt

a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z
```

```
cat test.txt | xargs

a b c d e f g h i j k l m n o p q r s t u v w x y z
```


-n选项多行输出,-d选项可以自定义一个定界符

```
echo "nameXnameXnameXname" | xargs -dX -n2

name name
name name
```

###  sort 排序
字段说明

 - -n 按数字进行排序 VS -d 按字典序进行排序
 - -r 逆序排序
 - -k N 指定按第N列排序

```
sort -nrk 1 data.txt
sort -bd data // 忽略像空格之类的前导空白字符
```

### uniq消除重复行
消除重复行

```
sort unsort.txt | uniq
```

统计各行在文件中出现的次数

```
sort unsort.txt | uniq -c
```

找出重复行

```
sort unsort.txt | uniq -d
```

### tr进行转换
tr用来进行字符转换


-d是删除，-c是求补集。

```
echo 12345 | tr '0-9' '9876543210' //加解密转换，替换对应字符
cat text| tr '\t' ' '  //制表符转空格

cat file | tr -d '0-9' // 删除所有数字
cat file | tr -c '0-9' //获取文件中所有数字

```

### cut按列切文本

```
cut -f2,4 filename  //截取文件的第2列和第4列
```


### wc统计行和字符

```
$wc -l file // 统计行数

$wc -w file // 统计单词数

$wc -c file // 统计字符数
```

### sed文本替换利器
sed是一种流编辑器，能够完美的配合正则表达式使用，功能不同凡响。

处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。**Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等**。

#### 替换操作：s命令

```
sed 's/book/books/' file  //替换文本中的字符串

sed 's/text/replace_text/g' file //全局替换(全面替换标记g)

sed '/^$/d' file //移除空白行
```


### awk数据流处理工具
awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是**把文件逐行的读入**，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

默认是用空格作为分隔符，如果要指定分割符，要用-F参数指定，比如`awk -F '|'`，表示用竖线来进行分割。


awk脚本结构

```
awk ' BEGIN{ statements } statements2 END{ statements } '
```

1. 执行begin中语句块；

2. 从文件或stdin中读入一行，然后执行statements2，重复这个过程，直到文件全部被读取完毕；

3. 执行end语句块；

且特殊变量：

```
NR:表示记录数量，在执行过程中对应当前行号；

NF:表示字段数量，在执行过程总对应当前行的字段数；

$0:这个变量包含执行过程中当前行的文本内容；

$1:第一个字段的文本内容；

$2:第二个字段的文本内容；
```

一下是以下范例：

```
[root@www ~]# last -n 5 <==仅取出前五行
root     pts/1   192.168.1.100  Tue Feb 10 11:21   still logged in
root     pts/1   192.168.1.100  Tue Feb 10 00:46 - 02:28  (01:41)
root     pts/1   192.168.1.100  Mon Feb  9 11:41 - 18:30  (06:48)
dmtsai   pts/1   192.168.1.100  Mon Feb  9 11:41 - 11:41  (00:00)
root     tty1                   Fri Sep  5 14:09 - 14:10  (00:01)

#last -n 5 | awk  '{print $1}'
root
root
root
dmtsai
root

awk ' END {print NR}' file //统计文件的行数

```

#### 合并奇偶行
awk可以调用getline手动读取下一行：

```
awk '{tmp=$0;getline;print tmp" "$0}'
```