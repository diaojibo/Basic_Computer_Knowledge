## C++字符串处理
记录一些C++字符串处理的常规操作

### 分割字符串
C++ 至今为止没有官方实现的字符串分割函数。相比 Python、Java 等语言，多少是有些不便的。

在C++中，分割可以用流的getline函数实现。它接收一个输入流，将输入流至行末/分隔符部分的字符串保存在临时的字符串中；同时，返回输入流的左值引用。考虑到输入流本身可以用作条件判断，我们可以将 std::getline 与 while 循环联用，达成目的。

``` c++
void split(const std::string& s,
    std::vector<std::string>& sv,
                   const char delim = ' ') {
    sv.clear();
    std::istringstream iss(s);
    std::string temp;

    while (std::getline(iss, temp, delim)) {
        sv.emplace_back(std::move(temp));
    }

    return;
}
```

代码中，我们借助字符串输入流 istringstream 处理带分割的字符串 s。而后将各个 delim 之间的内容，保存在临时字符串 temp 当中，并移动到向量 sv 的末尾。