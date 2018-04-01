# OSH实验一实验报告

作者：邓龙 学号：PB16110675

## 实验内容
请使用调试跟踪工具，追踪自己的操作系统（建议Linux）的启动过程，并找出其中至少两个关键事件。

## 实验环境
操作系统：Ubuntu 16.04
调试内核版本：4.15.14
编译器：gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9)
调试器：GNU gdb (GDB) 8.1
虚拟机：qemu 2.5.0

## 实验方法及过程

大致按照了参考文献[1]的内容。

### 编译内核

使用命令编译内核

    make menuconfig
    <save>
    make bzImage

使用dracut生成initramfs文件

    dracut init.img 4.15.14
    chmod 666 init.im

### qemu与gdb的配置
    
按参考文献[1]中方法修改gdb源码并编译，解决 g_packet 过长的问题

新建start.sh用作启动，代码如下

    #!/bin/bash
    qemu-system-x86_64 -kernel ./bzImage -initrd ./init.img -smp 2 -gdb tcp::1234 -m 1024 -append "console=ttyS0 earlyprintk=ttyS0,115200 nokaslr debug" -S

其中 console 和 earlyprink 参数配置了内核启动中早期过程中的串口输出，debug参数会使内核启动过程中额外输出一些信息

### 尝试开始调试

运行 ./start.sh 启动qemu
在新的终端中运行

    gdb vmlinux
    
启动gdb并导入 vmlinux中的符号表

在gdb中运行

    list
    info source

发现gdb在开始调试时已经进入 linux/init/header_64.S ，而符号表中能找到的运行最早的符号应为 start_kernel， 这给start_kernel之前的调试带来了困难。

## 参考文献

[1]  http://www.cnblogs.com/chineseboy/p/4216521.html 
[2]  https://www.gitbook.com/book/xinqiu/linux-insides-cn
