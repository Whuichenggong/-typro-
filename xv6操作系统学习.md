# xv6操作系统学习

---

**1.学会使用gdb调试**

**2.启动gdb服务器**

~~~
make CPUS=1 qemu-gdb
gdb mutiarch /xv6.../xv6.../kernel/kernel
file /xv6.../xv6.../kernel/kernel
target remote username@remotehost:port
~~~

**3.指令**

~~~
layout split
list main
关闭 layout split

ctrl-x o切换layout视图
layout src只显示源代码
layout asm 只显示汇编窗口
~~~

