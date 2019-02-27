# gdb调试

## 相关知识

### core dump

	当程序运行时异常崩溃时，操作系统会将程序当时的内存状态记录下来，保存在一个文件中，core dump 文件可以再现程序出错时的情景。
	
#### 产生core dump 的信号类型

| Signal    |  Action  |  Comment                                         |
| -------   |  :-----: |  :---------------------------------------:       |
| SIGQUIT   |   Core   |  Quit from keyboard                              |
| SIGILL    |   Core   |  Illegal Instruction                             |
| SIGABRT   |   Core   |  Abort signal from abort(程序调用 abort() 函数)  |
| SIGSEGV   |   Core   |  Invalid memory reference                        |
| SIGTRAP   |   Core   |  Trace/breakpoint trap                           |

在终端上按 ctrl + z 向进程发出 SIGTSTP 信号，ctrl + c 向进程发出 SIGINT 信号，kill -9 命令会发出 SIGKILL 命令, ctrl + \ 发出 SIGQUIT 命令

#### core dump 文件生成

-  在终端中输入ulimit -c 如果结果为0，说明当程序崩溃时，系统并不能生成core dump

- 使用ulimit -c unlimited命令，开启core dump功能，并且不限制生成core dump文件的大小。如果需要限制，加数字限制即可。ulimit - c 1024

- 用上面命令只会对当前的终端环境有效，如果想需要永久生效，可以修改文件 /etc/security/limits.conf文件

```

# /etc/security/limits.conf
#
#Each line describes a limit for a user in the form:
#
#<domain>   <type>   <item>   <value>
    *          soft     core   unlimited

```

- /proc/sys/kernel/core_uses_pid可以控制core文件的文件名中是否添加pid作为扩展。文件内容为1，表示添加pid作为扩展名，生成的core文件格式为core.xxxx；为0则表示生成的core文件同一命名为core
> echo "1" > /proc/sys/kernel/core_uses_pid

- /proc/sys/kernel/core_pattern可以控制core文件保存位置和文件名格式
> echo "/corefile/core-%e-%p-%t" > core_pattern  ( 可以将core文件统一生成到/corefile目录下，产生的文件名为core-命令名-pid-时间戳) 

```

%p - insert pid into filename 添加pid
%u - insert current uid into filename 添加当前uid
%g - insert current gid into filename 添加当前gid
%s - insert signal that caused the coredump into the filename 添加导致产生core的信号
%t - insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间
%h - insert hostname where the coredump happened into filename 添加主机名
%e - insert coredumping executable name into filename 添加命令名

```

## 编译可执行程序

- 编译时一定要注意加上 “-g” (将调试信息添加到可执行文件中，显示具体的函数名和变量)
> gcc -g test.c -o test

## 启动gdb

- gdb  program
> program 为带调试信息的可执行程序

- gdb program core
> core是程序core dump(异常错误)后产生的文件,program可执行文件和core一定要一一对应,即是可执行文件program
产生core文件.

- gdb program PID
> 调试已运行的程序

- gdb options(参数)

> -symbols file  == -s file 从指定文件中读取符号表  
> -se file  从指定文件中读取符号表并将它使用在可执行文件中  
> -c file == -core file 使用该文件作为 core dump   
> -directory directory == -d directory  加入一个源文件的搜索路径。默认搜索路径是环境变量中PATH所定义的路径  

## 问题解决

- 如果在gdb中出现??()

```shell
    1.考虑是否加 -g 
    2. # gdb --quiet -nx --readnever /proc/$pid/exe $pid
       安装对应的依赖软件: # debuginfo-install glibc-2.17-157.el7_3.5.x86_64

```

## gdb相关操作

### 查看源代码

- (gdb) l linenum                   <-----显示程序第linenum行周围的源程序

- (gdb) l function                  <-----显示函数名为function的函数的源程序

- (gdb) l                           <-----显示当前行后面的源程序

- (gdb) l -                         <-----显示当前行前面的源程序

### 设置断点（BreakPoint）

- (gdb) b 16                   <-----设置断点，在源程序第16行处。

- (gdb) b filename:linenum     <-----在源文件filename的linenum行处停住。

- (gdb) b func                 <-----设置断点 可以是函数名，在函数func()入口处。

- (gdb) b filename:function    <-----在源文件filename的function函数的入口处停住。

- (gdb) b ... if condition     <----- ...可以是上述的参数，condition表示条件，在条件成立时停住。

- (gdb) info b                 <-----查看断点信息。

```

Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x000000000040055c in main at test.c:16
2       breakpoint     keep y   0x000000000040052d in func at test.c:5

```

- (gdb) d 1        <------- 删除1号断点


### 设置观察点（WatchPoint）

观测点用来观察某一表达式（变量）是否发生变化，如果有变化，马上停住程序,**在run之前直接设置观察点是不行的,要在运行程序之后设置watch**

- (gdb) watch expr            <------- 为表达式（变量）expr设置一个观察点。一旦表达式值有变化时，马上停住程序。

- (gdb) rwatch expr           <------- 当表达式（变量）expr被读时，停住程序。

- (gdb) awatch expr           <------- 当表达式（变量）的值被读或被写时，停住程序。

- (gdb) info watchpoints      <------- 列出当前所设置了的所有观察点。。

### 管理停止点

- (gdb) clear                                <------- 清除所有的停止点（包括breakpoints,watchpoints）

- (gdb) clear filename:function              <------- 清除该函数名那一行所有的停止点（包括breakpoints,watchpoints）

- (gdb) clear filename:linenum               <------- 清除指定行的所有停止点

- (gdb) disable [breakpoints] [range...]     <------- 使指定的停止点不工作，但不删除

- (gdb) enable [breakpoints] [range...]      <------- 使能指定的停止点

### 查看堆栈信息

- (gdb) bt                <------- 查看当前程序运行到哪里，函数堆栈。

- (gdb) info locals       <------- 打印当前函数所有局部变量的值

- (gdb) info args         <------- 打印当前函数传入参数的值

- (gdb) f n               <------- 根据bt命令确定各个堆栈的序号，可以切换到指定的堆栈(函数中)，frame

### 线程调试

- (gdb) info thread                                  <------- 查看线程信息

- (gdb) thread 序号                                   <------- 切换到对应的线程 ,序号由 (gdb) info thread 命令获得 掉
                                                               或则由 $ pstack pid 获得
- (gdb) set scheduler-locking [off|on|step]          <------- 调用多线程程序时, 默认是所有多个线程同时运行, 像只让调试线程
                                                              运行,　off: 不锁定任何线程, 所有的线程都继续执行
                                                              on: 只有当前被调试线程会继续执行
                                                              step: 单步执行时, 只有当前线程会被执行
                                                   

### 常用命令

- (gdb) set listsize num         <------- 设置一次显示源代码的行数

- (gdb) show listsize            <------- 显示一次显示源代码的行数

- (gdb) dir dirname1:dirname2    <------- 加入源文件的绝对路径(可以是目录)到gdb搜索范围内 directory

- (gdb) directory                <------- 清除所有的自定义的源文件搜索路径信息

- (gdb) r                        <------- 程序运行

- (gdb) n                        <------- 单条语句执行，如果遇到函数**不进入函数**next命令简写。

- (gdb) s                        <------- 单步调试，如果遇到函数**进入函数**(step)

- (gdb) u                        <------- 当你厌倦了在一个循环体内单步跟踪时，退出循环体(前提是该循环内没有停止点) until

- (gdb) c                        <------- 继续运行程序直到下一个断点或程序结束，continue命令简写。

- (gdb) p i                      <------- 打印变量i的值，print命令简写。

- (gdb) p/format i               <------- 以特定的格式打印变量i的值，例如 p/x i (以十六进制打印i)

- (gdb) p *array@len             <------- 打印动态数组的内容，array首地址，len为长度。


- (gdb) finish                   <------- 当运行到函数中时，查看函数的返回值通过finish

  Run till exit from #0  func (n=250) at test.c:5  
  0x00000000004005a0 in main () at test.c:24  
  24         printf("result[1-250] = %d \n", func(250) );  
  **Value returned is $10 = 31125**
  
- (gdb) where                            <------- 查看当前程序调用函数栈跟bt差不多
                                        
- (gdb) shell command                    <------- 可以在gdb界面中运行shell 命令 如：shell ls

- (gdb) set args argc[1] argc[2] ....    <------- 设置程序运行时传入的参数 如：set args 10 

- (gdb) path  dir                        <------- 设置程序的运行路径

- (gdb) show paths                       <------- 查看程序的运行路径。

### 实例一

```shell

    1.gdb进入已经运行的进程
        $ gdb ./lock 28150
        
    2.查看该进程的线程信息
        (gdb) info thread 
        结果:
             Id   Target Id         Frame 
              5    Thread 0x7ff36ffec700 (LWP 28151) "lock" __lll_lock_wait ()
                at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
              4    Thread 0x7ff36f7eb700 (LWP 28152) "lock" __lll_lock_wait ()
                at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
              3    Thread 0x7ff36efea700 (LWP 28153) "lock" 0x00007ff3700ab66d in nanosleep ()
                at ../sysdeps/unix/syscall-template.S:81
              2    Thread 0x7ff36e7e9700 (LWP 28154) "lock" 0x00007ff3700ab66d in nanosleep ()
                at ../sysdeps/unix/syscall-template.S:81
            * 1    Thread 0x7ff370feb740 (LWP 28150) "lock" 0x00007ff370bd7ef7 in pthread_join (
                threadid=140683532748544, thread_return=0x0) at pthread_join.c:92
                
         其中 LWP 28151 代表线程的唯一码.
         
    3.切换到特定的线程
        (gdb) thread 5
        
    4.查看这时线程调用函数栈
        (gdb) where
        
        结果:
            #0  __lll_lock_wait () at ../nptl/sysdeps/unix/sysv/linux/x86_64/lowlevellock.S:135
            #1  0x00007ff370bd8d02 in _L_lock_791 () from /lib64/libpthread.so.0
            #2  0x00007ff370bd8c08 in __GI___pthread_mutex_lock (mutex=0x6020e0 <mutex2>) at pthread_mutex_lock.c:64
            #3  0x00000000004008ce in func1 () at lock.cpp:18
            #4  0x0000000000400966 in thread1 (arg=0x0) at lock.cpp:43
            #5  0x00007ff370bd6dc5 in start_thread (arg=0x7ff36ffec700) at pthread_create.c:308
            #6  0x00007ff3700e476d in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:113
            
    5.进入某一个特定的函数栈
        (gdb) f 3
        
        结果:
            #3  0x00000000004008ce in func1 () at lock.cpp:18
            18          pthread_mutex_lock(&mutex2); 
            
            
    6.打印相关的变量值
        (gdb) p mutex2
        
        结果:
            $2 = {__data = {__lock = 2, __count = 0, __owner = 28152, __nusers = 1, __kind = 0, __spins = 0, __list = {
                  __prev = 0x0, __next = 0x0}}, 
              __size = "\002\000\000\000\000\000\000\000\370m\000\000\001", '\000' <repeats 26 times>, __align = 2}
              
        这里 __owner = 28152 代表 mutex2已经被线程4占用(LWP 28152)
        

```

### 实例二

```shell
    调试死锁问题
        1. 运行可执行程序
                > ./sample_test
                
        2. 查看 sample_test 的进程号
                > ps aux | grep "sample_test"
                
        3. 运行 gdb 程序,  再 attach 进程号, 再　thread apply all bt
                结果:
                    > gdb
                    (gdb) attach 进程号
                    (gdb) thread apply all bt
```
[参考资料](https://www.ibm.com/developerworks/cn/linux/l-cn-deadlock/)