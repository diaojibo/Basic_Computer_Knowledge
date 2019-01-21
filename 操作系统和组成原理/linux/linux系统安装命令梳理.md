## 系统安装命令梳理
笔记梳理一下不同场景下遇到的安装命令


### yum
 - 全称是“Yellow dog Updater,Modified”
 - 是redhat、centos、Fedora系统(都是基于linux的系统)下的安装方式
 - 能够指定服务器自动下载RPM包并安装（也可设置只下载包但不安装），可以自动处理依赖关系，安装所有依赖的软件包。


### rpm
 - 全称是“redhat package management”
 - redhat系统下的软件管理包
 - 用来安装、卸载xx.rpm软件

使用wget下载一个 rpm包, 然后用 rpm -ivh  xxx.rpm  安装这个包，系统可能会提示在安装此包之前，必须先安装另一个包，只有将那个包安装好，才能继续安装你要安装的包，这种就是包之间的依赖关系。

如果安装一个包，还要依赖好多包，那rpm就不太好用了，这种情况下，通常使用yum命令。  yum  install  xxx(包名)，能自动下载和安装依赖包。

### apt-get
是ubuntu下的一种安装方式，基于debain