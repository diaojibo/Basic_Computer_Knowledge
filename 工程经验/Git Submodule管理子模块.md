## git子模块
git项目中通常要使用子模块来引用一些公共的项目。

git Submodule 是一个很好的多项目使用共同类库的工具，他允许类库项目做为repository,子项目做为一个单独的git项目存在父项目中，子项目可以有自己的独立的commit，push，pull。而父项目以Submodule的形式包含子项目，父项目可以指定子项目header，父项目中会的提交信息包含Submodule的信息，再clone父项目的时候可以把Submodule初始化。

### 添加submodule
使用git命令可以直接添加Submodule：

```
git submodule add git@github.com:jjz/pod-library.git pod-library
```

用git status命令就可以看到

```
git status
```

```
    On branch master
    Changes to be committed:
    
        new file:   .gitmodules
        new file:   pod-library
```

.gitmodules 内容包含Submodule的主要信息，指定reposirory

```
[submodule "pod-library"]
        path = pod-library
        url = git@github.com:jjz/pod-library.git
```

### clone submodule
有两种方式把子项目的代码全都copy下来，一种是是先clone父项目，再clone子项目。一种是直接在clone的时候就添加recursively指令

简而言之，一种就是直接：

```
git clone git@github.com:jjz/pod-project.git --recursive
```

一种就是clone父项目以后再打两个命令：

```
git submodule init
git submodule update
```

### 用户名密码设定
以免每次都要你重新输入git的用户名密码，可以在配置中加上下面配置：

```
git config --global credential.helper store
```