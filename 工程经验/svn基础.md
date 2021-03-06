## svn基础
与Git相对,Subversion(SVN) 是一个开源的版本控制系統.

数据放置在一个中央资料档案库(repository) 中。 这个档案库很像一个普通的文件服务器, 不过它会记住每一次文件的变动。 这样你就可以把档案恢复到旧的版本, 或是浏览文件的变动历史。

一些概念：

 - repository（源代码库）:源代码统一存放的地方
 - Checkout（提取）:当你手上没有源代码的时候，你需要从repository checkout一份
 - Commit（提交）:当你已经修改了代码，你就需要Commit到repository
 - Update (更新):当你已经Checkout了一份源代码， Update一下你就可以和Repository上的源代码同步，你手上的代码就会有最新的变更

日常开发过程其实就是这样的（假设你已经Checkout并且已经工作了几天）：Update(获得最新的代码) -->作出自己的修改并调试成功 --> Commit(大家就可以看到你的修改了) 。

如果两个程序员同时修改了同一个文件呢, SVN可以合并这两个程序员的改动，实际上SVN管理源代码是以行为单位的，就是说两个程序员只要不是修改了同一行程序，SVN都会自动合并两种修改。如果是同一行，SVN会提示文件Confict, 冲突，需要手动确认。

### 常用命令

更新到最新版本的仓库代码：

```
svn up
```

查看当前仓库的历史代码版本：

```
svn log
```

提交代码：

```
svn ci -m "someth"
```

本地暂时回退仓库到某一历史版本：

```
svn up -r r123412
```

看当前目录地址信息：

```
svn info .
```

### 解决冲突
当你从版本库更新、合并文件时，或者切换工作副本至一个不同的 URL 时你会遇到冲突。有两种冲突:

 - 文件冲突,当两名(或更多)开发人员修改了同一个文件中相邻或**相同的行**时就会发生文件冲突。

 - 树冲突,当一名开发人员移动、重命名、删除一个文件或文件夹，而另一名开发人员也对它们进行了移动、重命名、删除或者仅仅是修改时就会发生树冲突。