## pid
在linux系统的目录/var/run下面一般我们都会看到很多的*.pid文件。而且往往新安装的程序在运行后也会在/var/run目录下面产生自己的pid文件。那么这些pid文件有什么作用呢？它的内容又是什么呢？


实质上pid的作用比较好理解：

 - pid文件的内容：pid文件为**文本文件**，内容只有一行, 记录了该进程的ID。 
用cat命令可以看到。 
 - pid文件的作用：防止进程启动多个副本。只有获得pid文件(固定路径固定文件名)写入权限(F_WRLCK)的进程才能正常启动并把自身的PID写入该文件中。其它同一个程序的多余进程则自动退出。 


## 参考
[/unix下 pid文件作用浅析](https://siwind.iteye.com/blog/1753517)