# gdb调试

- 编译时一定要注意加上 “-g” 
> gcc -g test.c -o test

## gdb相关操作

- (gdb) b 16    <----- 设置断点，在源程序第16行处。

- (gdb) b func  <--------设置断点，在函数func()入口处。

- (gdb) info b    <------- 查看断点信息。

```

Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x000000000040055c in main at test.c:16
2       breakpoint     keep y   0x000000000040052d in func at test.c:5

```

- (gdb) d 1    <------- 删除1号断点