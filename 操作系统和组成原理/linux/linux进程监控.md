## linux进程监控相关

### ps
Linux中的ps命令是Process Status的缩写。ps命令用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，就是执行ps命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用top命令。

linux上进程有5种状态:

 - 运行(正在运行或在运行队列中等待)
 - 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
 - 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
 - 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
 - 停止(进程收到SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU信号后停止运行运行)

ps工具标识进程的5种状态码: 
 - D 不可中断 uninterruptible sleep (usually IO) 
 - R 运行 runnable (on run queue) 
 - S 中断 sleeping 
 - T 停止 traced or stopped 
 - Z 僵死 a defunct (”zombie”) process 



```
ps -ef // 显示所有进程信息，连同命令行
```

### pstack 跟踪进程栈
此命令可显示每个进程的栈跟踪。pstack 命令必须由相应进程的属主或 root 运行。可以使用 pstack 来确定进程挂起的位置。此命令允许使用的唯一选项是要**检查的进程的 PID**。

这个命令在排查进程问题时非常有用，比如我们发现一个服务一直处于work状态（如假死状态，好似死循环），使用这个命令就能轻松定位问题所在；可以在一段时间内，多执行几次pstack，若发现代码栈总是停在同一个位置，那个位置就需要重点关注，很可能就是出问题的地方；

示例：查看bash程序进程栈:

```
/opt/app/tdev1$ps -fe| grep bash
tdev1   7013  7012  0 19:42 pts/1    00:00:00 -bash
tdev1  11402 11401  0 20:31 pts/2    00:00:00 -bash
tdev1  11474 11402  0 20:32 pts/2    00:00:00 grep bash
/opt/app/tdev1$pstack 7013
#0  0x00000039958c5620 in __read_nocancel () from /lib64/libc.so.6
#1  0x000000000047dafe in rl_getc ()
#2  0x000000000047def6 in rl_read_key ()
#3  0x000000000046d0f5 in readline_internal_char ()
#4  0x000000000046d4e5 in readline ()
#5  0x00000000004213cf in ?? ()
#6  0x000000000041d685 in ?? ()
#7  0x000000000041e89e in ?? ()
#8  0x00000000004218dc in yyparse ()
#9  0x000000000041b507 in parse_command ()
#10 0x000000000041b5c6 in read_command ()
#11 0x000000000041b74e in reader_loop ()
#12 0x000000000041b2aa in main ()
```


### strace 跟踪进程中的系统调用
strace常用来跟踪进程执行时的系统调用和所接收的信号。 在Linux世界，进程不能直接访问硬件设备，当进程需要访问硬件设备(比如读取磁盘文件，接收网络数据等等)时，必须由用户态模式切换至内核态模式，通过系统调用访问硬件设备。strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。

#### 输出参数含义
每一行都是一条系统调用，等号左边是系统调用的函数名及其参数，右边是该调用的返回值。 strace 显示这些调用的参数并返回符号形式的值。strace 从内核接收信息，而且不需要以任何特殊的方式来构建内核。

```
$strace cat /dev/null
execve("/bin/cat", ["cat", "/dev/null"], [/* 22 vars */]) = 0
brk(0)                                  = 0xab1000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f29379a7000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
...
```

### ipcs 查询进程间通信状态
ipcs是Linux下显示进程间通信设施状态的工具。可以显示消息队列、共享内存和信号量的信息。对于程序员非常有用，普通的系统管理员一般用不到此指令。

#### 查看系统使用的IPC资源

```
$ipcs

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x00000000 229376     weber      600        1

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
```

分别查询IPC资源:

```
$ipcs -m 查看系统使用的IPC共享内存资源
$ipcs -q 查看系统使用的IPC队列资源
$ipcs -s 查看系统使用的IPC信号量资源
```

### top linux下的任务管理器
top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。top是一个动态显示过程,即可以通过用户按键来不断刷新当前状态.

如果在前台执行该命令,它将**独占前台**,直到用户终止该程序为止.比较准确的说,top命令提供了实时的对系统处理器的状态监视.它将显示系统中CPU最“敏感”的任务列表.

该命令可以按**CPU使用.内存使用和执行时间对任务进行排序**；而且该命令的很多特性都可以通过交互式命令或者在个人定制文件中进行设定。

```
$top
    top - 09:14:56 up 264 days, 20:56,  1 user,  load average: 0.02, 0.04, 0.00
    Tasks:  87 total,   1 running,  86 sleeping,   0 stopped,   0 zombie
    Cpu(s):  0.0%us,  0.2%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.2%st
    Mem:    377672k total,   322332k used,    55340k free,    32592k buffers
    Swap:   397308k total,    67192k used,   330116k free,    71900k cached
    PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    1 root      20   0  2856  656  388 S  0.0  0.2   0:49.40 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0     0    0    0 S  0.0  0.0   7:15.20 ksoftirqd/0
    4 root      RT   0     0    0    0 S  0.0  0.0   0:00.00 migration/0
```

**第一行**
 - 09:14:56 ： 系统当前时间
 - 264 days, 20:56 ： 系统开机到现在经过了多少时间
 - 1 users ： 当前2用户在线
 - load average: 0.02, 0.04, 0.00： 系统1分钟、5分钟、15分钟的CPU负载信息


**第二行**
Tasks：任务;
 - 87 total：很好理解，就是当前有87个任务，也就是87个进程。
 - 1 running：1个进程正在运行
 - 86 sleeping：86个进程睡眠
 - 0 stopped：停止的进程数
 - 0 zombie：僵死的进程数

**第三行**
 - Cpu(s)：表示这一行显示CPU总体信息
 - 0.0%us：用户态进程占用CPU时间百分比，不包含renice值为负的任务占用的CPU的时间。
 - 0.7%sy：内核占用CPU时间百分比
 - 0.0%ni：改变过优先级的进程占用CPU的百分比
 - 99.3%id：空闲CPU时间百分比
 - 0.0%wa：等待I/O的CPU时间百分比
 - 0.0%hi：CPU硬中断时间百分比
 - 0.0%si：CPU软中断时间百分比
 - 注：这里显示数据是所有cpu的平均值，如果想看每一个cpu的处理情况，按1即可；折叠，再次按1；


**第四行**
  - Men：内存的意思
  - 8175320kk total：物理内存总量
  - 8058868k used：使用的物理内存量
  - 116452k free：空闲的物理内存量
  - 283084k buffers：用作内核缓存的物理内存量

**第五行**
  - Swap：交换空间
  - 6881272k total：交换区总量
  - 4010444k used：使用的交换区量
  - 2870828k free：空闲的交换区量
  - 4336992k cached：缓冲交换区总量

**进程信息**
  - 再下面就是进程信息：
  - PID：进程的ID
  - USER：进程所有者
  - PR：进程的优先级别，越小越优先被执行
  - NInice：值
  - VIRT：进程占用的虚拟内存
  - RES：进程占用的物理内存(应该说是常驻内存的大小)
  - SHR：进程使用的共享内存
  - S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
  - %CPU：进程占用CPU的使用率
  - %MEM：进程使用的物理内存和总内存的百分比
  - TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
  - COMMAND：进程启动命令名称


#### top命令交互操作指令
使用频率最高的是P、T、M，因为通常使用top，我们就想看看是哪些进程最耗cpu资源、占用的内存最多； 注：通过”shift + >”或”shift + <”可以向右或左改变排序列 如果只需要查看内存：可用free命令。只查看uptime信息（第一行），可用uptime命令；

 - P：按%CPU使用率排行
 - T：按MITE+排行
 - M：按%MEM排行


在top基本视图中，按键盘数字“1”，可监控每个逻辑CPU的状况；


#### 面试题：cpu 100%找原因
先用top查看是哪些进程的cpu占用率不正常
然后用lsof查看这个进程打开了哪些文件，用netstat查看这个进程有哪些网络连接

通过流量监控工具看流量ntop
看系统资源，MEM,CPU,DISK I/O ：vmstat,free.
