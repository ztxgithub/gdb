# gdb调试

- 编译时一定要注意加上 “-g” 
> gcc -g test.c -o test

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

```

	Run till exit from #0  func (n=250) at test.c:5
	0x00000000004005a0 in main () at test.c:24
	24         printf("result[1-250] = %d \n", func(250) );
	**Value returned is $10 = 31125**

```