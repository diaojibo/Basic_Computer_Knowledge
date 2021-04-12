## 测试框架test::nginx

test::nginx 是openresty的一个测试框架，使用perl语言编写，测试用例使用DSL语法，可以穿插perl脚本。



安装：先安装cpanminus

```
curl -L https://cpanmin.us | perl - --sudo App::cpanminus
# or
curl -L https://cpanmin.us | perl - App::cpanminus
```

然后

```
cpanm --notest Test::Nginx IPC::Run
```

