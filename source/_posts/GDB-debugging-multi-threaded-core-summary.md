---
title: gdb调试多线程core总结
date: 2019-03-02 17:06:20
tags: linux
---

#### 起源
最近在编写代码时遇到一个很诡异的问题，简单的多线程，但是却core在了“printf”语句和atomic语句，这个明显是不合理的。所以试图用gdb进行多线程调试，找出问题所在。

#### 简单介绍
先介绍一下GDB多线程调试的基本命令。

1. `info threads` 显示当前可调试的所有线程，每个线程会有一个GDB为其分配的ID，后面操作线程的时候会用到这个ID。 前面有*的是当前调试的线程。

2. `thread ID` 切换当前调试的线程为指定ID的线程。
注意：切换到不同的线程id，使用bt可以打印该id的堆栈信息。

3. `break thread_test.c:123 thread all` 在所有线程中相应的行上设置断点

4. `thread apply ID1 ID2 command` 让一个或者多个线程执行GDB命令command。

5. `thread apply all command` 让所有被调试线程执行GDB命令command。
如：`thread apply all bt`打印所有线程的堆栈信息。

6. `set scheduler-locking off|on|step`：估计是实际使用过多线程调试的人都可以发现，在使用step或者continue命令调试当前被调试线程的时候，其他线程也是同时执行的，怎么只让被调试程序执行呢？通过这个命令就可以实现这个需求。off 不锁定任何线程，也就是所有线程都执行，这是默认值。 on 只有当前被调试程序会执行。 step 在单步的时候，除了next过一个函数的情况(熟悉情况的人可能知道，这其实是一个设置断点然后continue的行为)以外，只有当前线程会执行。

7. 使用gdb运行带参数的二进制：例如一个程序名为`prog` 参数为 `-l a -C abc`，使用`gdb prog`之后，在gdb界面输入`set args -l a -C abc`，后面就可以`r`了。

8. 查看汇编指令`disassemble`，往往需要配合查看寄存器信息`info register`和栈信息`info frame 0`

9. 修改输出core的位置：
`/proc/sys/kernel/core_pattern`，可以修改为`echo "/corefile/core-%e-%p-%t" > /proc/sys/kernel/core_pattern`

