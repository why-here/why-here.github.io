为了能生成 core file，需要先设置 ulimit 中 core file size。默认为 0，即不保存 core file。

```shell
$ ulimit -a # 查看限制
$ ulimit -c unlimited # 不限制大小
```

为了更方便地调试，可以通过 `g++ -g file.cpp` 命令，将一些符号信息和行信息添加进文件中。

#### 开启 gdb 的三种方法：

参考 man 手册

- gdb program
- gdb program core ，发生 core dump 时
- gdb program 1234 or gdb -p 1234，可以调试正在运行的程序

#### 常用的 gdb 命令：

- break [file:]function ，在 file 文件（可选）的 function 函数处设置断点；break 20 ,在第 20 行设置断点；
- 条件断点：break [where] if [condition]
- run [arglist] ： 以 arglist 为参数（可选）运行程序
- bt ：打印程序调用栈
- print expr or p expr ： 打印变量或者表达式的值，p *d 可打印指针 d 指向的内容。
- x 0x..... : 查看内存的内容。
- c ：继续运行程序
- next or n ：执行下一行程序，跳过函数详细调用过程
- step or s ：执行下一行程序，并进入函数（若有的话）。
- list : 显示当前停止处的源代码，或 list 20,显示 20 行的代码，或 list func ，显示函数 func 的代码。

#### 多线程调试： 

参考 [coolshell](https://coolshell.cn/articles/3643.html)

- info thread 查看当前进程的线程。
- thread <ID> 切换调试的线程为指定ID的线程。
- break file.c:100 thread all  在file.c文件第100行处为所有经过这里的线程设置断点。
- set scheduler-locking off|on|step，这个是问得最多的。在使用step或者continue命令调试当前被调试线程的时候，其他线程也是同时执行的，怎么只让被调试程序执行呢？通过这个命令就可以实现这个需求。
    - off 不锁定任何线程，也就是所有线程都执行，这是默认值。
    - on 只有当前被调试程序会执行。
    - step 在单步的时候，除了next过一个函数的情况(熟悉情况的人可能知道，这其实是一个设置断点然后 continue 的行为)以外，只有当前线程会执行。

一个简单的单线程段错误调试例子，可以参考 [gilpin](http://www.cs.cmu.edu/~gilpin/tutorial/).

更详细的 gdb 命令中文解释可以参考博文 [CSDN](https://blog.csdn.net/bigheadyushan/article/details/77828949).