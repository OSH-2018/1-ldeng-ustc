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
    
发现gdb在开始调试时已经进入 linux/init/head_64.S ，而符号表中能找到的运行最早的符号应为 start_kernel， 这给start_kernel之前的调试带来了困难。
考虑到除去引导部分，内核启动时应从linux/arch/x86/boot/head.S 开始，而第一个C语言代码在 arch/x86/boot/main.c 阅读代码，在拷贝参数后，早期的控制台已经启动，而通过gdb在start_kernel处设置断点，观察控制台输出，有

    early console in setup code

这是main.c中两行代码的结果，在qemu启动中，我们给命令行加入了debug参数，早期控制台初始化完毕后输出了信息。

    if (cmdline_find_option_bool("debug"))
        puts("early console in setup code\n");
        
故从此行之后可利用puts即时向控制台输出信息进行调试。

### 关键事件1

arch/x86/boot/main.c 中 main 函数开始运行即为第一个关键事件。
/boot/main.c 是系统运行中第一个运行的C语言函数。各种设备的初始化基本都是在此执行的，包括

* 1 控制台初始化，这也是我们能够进行调试的基础
* 2 


## 参考文献

[1]  http://www.cnblogs.com/chineseboy/p/4216521.html 
[2]  https://www.gitbook.com/book/xinqiu/linux-insides-cn
