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

- proc/sys/kernel/core_pattern可以控制core文件保存位置和文件名格式
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
> core是程序core dump(异常错误)后产生的文件。

- gdb program PID
> 调试已运行的程序

- gdb options(参数)

> -symbols file  == -s file 从指定文件中读取符号表  
> -se file  从指定文件中读取符号表并将它使用在可执行文件中  
> -c file == -core file 使用该文件作为 core dump   
> -directory directory == -d directory  加入一个源文件的搜索路径。默认搜索路径是环境变量中PATH所定义的路径  

## gdb相关操作

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

观测点用来观察某一表达式（变量）是否发生变化，如果有变化，马上停住程序

- (gdb) watch expr            <-------  为表达式（变量）expr设置一个观察点。一旦表达式值有变化时，马上停住程序。

- (gdb) rwatch expr           <------- 当表达式（变量）expr被读时，停住程序。

- (gdb) awatch expr           <------- 当表达式（变量）的值被读或被写时，停住程序。

- (gdb) info watchpoints      <------- 列出当前所设置了的所有观察点。。






- (gdb) n          <------- 单条语句执行，next命令简写。

- (gdb) n          <------- 单条语句执行，next命令简写。

- (gdb) c          <------- 继续运行程序直到下一个断点，continue命令简写。

- (gdb) p i        <------- 打印变量i的值，print命令简写。

- (gdb) bt         <------- 查看当前程序运行到哪里，函数堆栈。

- (gdb) finish     <------- 当运行到函数中时，查看函数的返回值通过finish

  Run till exit from #0  func (n=250) at test.c:5  
  0x00000000004005a0 in main () at test.c:24  
  24         printf("result[1-250] = %d \n", func(250) );  
  **Value returned is $10 = 31125**
  
- (gdb) where                            <------- 查看当前程序调用函数栈跟bt差不多
                                        
- (gdb) shell command                    <------- 可以在gdb界面中运行shell 命令 如：shell ls

- (gdb) set args argc[1] argc[2] ....    <------- 设置程序运行时传入的参数 如：set args 10 

- (gdb) path  dir                        <------- 设置程序的运行路径

- (gdb) show paths                       <------- 查看程序的运行路径。
