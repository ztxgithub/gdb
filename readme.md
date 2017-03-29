# gdb调试

## 相关知识

### core dump

	当程序运行时异常崩溃时，操作系统会将程序当时的内存状态记录下来，保存在一个文件中，core dump 文件可以再现程序出错时的情景。
	
#### 产生core dump 的信号类型

| Signal    |  Action  |  Comment                                         |
| -------   |  :-----: |  :---------------------------------------:       |
| SIGQUIT   |   Core   |  Quit from keyboard (ctrl + \)                   |
| SIGILL    |   Core   |  Illegal Instruction                             |
| SIGABRT   |   Core   |  Abort signal from abort(程序调用 abort() 函数)  |
| SIGSEGV   |   Core   |  Invalid memory reference                        |
| SIGTRAP   |   Core   |  Trace/breakpoint trap                           |

## 编译可执行程序

- 编译时一定要注意加上 “-g” (将调试信息添加到可执行文件中，显示具体的函数名和变量)
> gcc -g test.c -o test

## 启动gdb

- gdb <program> 
> program 为带调试信息的可执行程序

- gdb <program> core
> core是程序core dump(异常错误)后产生的文件。

- gdb options(参数)
> -symbols <file>  == -s <file> 从指定文件中读取符号表
> -se <file>  从指定文件中读取符号表并将它使用在可执行文件中
> -c <file> == -core <file> 使用该文件作为 core dump 

## gdb相关操作

- (gdb) b 16     <-----设置断点，在源程序第16行处。

- (gdb) b func   <-----设置断点 可以是函数名，在函数func()入口处。

- (gdb) info b   <-----查看断点信息。

```

Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x000000000040055c in main at test.c:16
2       breakpoint     keep y   0x000000000040052d in func at test.c:5

```

- (gdb) d 1        <------- 删除1号断点

- (gdb) n          <------- 单条语句执行，next命令简写。

- (gdb) c          <------- 继续运行程序直到下一个断点，continue命令简写。

- (gdb) p i        <------- 打印变量i的值，print命令简写。

- (gdb) bt         <------- 查看当前程序运行到哪里，函数堆栈。

- (gdb) finish         <------- 当运行到函数中时，查看函数的返回值通过finish

  Run till exit from #0  func (n=250) at test.c:5  
  0x00000000004005a0 in main () at test.c:24  
  24         printf("result[1-250] = %d \n", func(250) );  
  **Value returned is $10 = 31125**
