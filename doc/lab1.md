# Lab1

## Part 1: PC Bootstrap
第一部分不需要写代码，只是通过QEMU和GDB调试，了解启动过程。回答一些问题

### Getting Started with x86 assembly

[X86汇编参考手册](https://pdos.csail.mit.edu/6.828/2018/readings/pcasm-book.pdf). X86汇编的语法格式有Intel和AT&T两种，上述参考手册中使用的是NASM汇编器（Intel）语法，而GNU的汇编器使用的是AT&T语法。我们使用GCC编译内核代码，所以在C语言中内嵌汇编通常使用AT&T格式的， [Brennan's Guide to Inline Assembly](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html)


**Exercise1**:
> 根据[参考资料](https://pdos.csail.mit.edu/6.828/2018/reference.html)熟悉X86的汇编，再后续的学习中会经常需要使用的参考页面提供的资料

### Simulating the x86
使用QEMU模拟器开发一个操作系统，QEMU内置的输出窗口调试功能有限，我们可以使用GDB远程连接QEMU进行调试，具体操作如下：

```bash
git clone https://pdos.csail.mit.edu/6.828/2018/jos.git lab 
cd lab
make
+ as kern/entry.S
+ cc kern/entrypgdir.c
+ cc kern/init.c
+ cc kern/console.c
+ cc kern/monitor.c
+ cc kern/printf.c
+ cc kern/kdebug.c
+ cc lib/printfmt.c
+ cc lib/readline.c
+ cc lib/string.c
+ ld obj/kern/kernel
+ as boot/boot.S
+ cc -Os boot/main.c
+ ld boot/boot
boot block is 382 bytes (max 510)
+ mk obj/kern/kernel.img
```
编译后会生成磁盘镜像文件 (`obj/kern/kernel.img`)，启动QEMU时会加载改文件。这个镜像文件包含了bootloader（`obj/boot/boot`） 和内核（`obj/kernel`）.

启动qemu
```bash
make qemu
```
或
```bash
make qemu-nox
```
这两个命令的区别，前者会有一个独立的qemu终端窗口出现，而后者没有独立的终端窗口，信息显示在当前的终端中。
这个内核目前只有两条命令，`help` 和 `keninfo` , 使用 `Ctrl+a, x`退出
```bash
sed "s/localhost:1234/localhost:26000/" < .gdbinit.tmpl > .gdbinit
***
*** Use Ctrl-a x to exit qemu
***
qemu-system-i386 -nographic -drive file=obj/kern/kernel.img,index=0,media=disk,format=raw -serial mon:stdio -gdb tcp::26000 -D qemu.log 
6828 decimal is XXX octal!
entering test_backtrace 5
entering test_backtrace 4
entering test_backtrace 3
entering test_backtrace 2
entering test_backtrace 1
entering test_backtrace 0
leaving test_backtrace 0
leaving test_backtrace 1
leaving test_backtrace 2
leaving test_backtrace 3
leaving test_backtrace 4
leaving test_backtrace 5
Welcome to the JOS kernel monitor!
Type 'help' for a list of commands.
K> help
help - Display this list of commands
kerninfo - Display information about the kernel
K> kerninfo
Special kernel symbols:
  _start                  0010000c (phys)
  entry  f010000c (virt)  0010000c (phys)
  etext  f010178e (virt)  0010178e (phys)
  edata  f0119300 (virt)  00119300 (phys)
  end    f0119940 (virt)  00119940 (phys)
Kernel executable memory footprint: 103KB
K> QEMU: Terminated
```


内核基本信息：
| 描述 | 虚拟地址 | 物理地址 |
| --- | --- | --- | 
| _start |  | 0010000c|
| entry | f010000c | 0010000c |
| etext | f0119300 | 00119300 |
| edata | f010000c | 0010000c |
| end | f0119940 | 00119940 |


### The PC's Physical Address Space
32bit(4GB) 物理内存布局
```text
+------------------+  <- 0xFFFFFFFF (4GB)
|      32-bit      |
|  memory mapped   |
|     devices      |
|                  |
/\/\/\/\/\/\/\/\/\/\

/\/\/\/\/\/\/\/\/\/\
|                  |
|      Unused      |
|                  |
+------------------+  <- depends on amount of RAM
|                  |
|                  |
| Extended Memory  |
|                  |
|                  |
+------------------+  <- 0x00100000 (1MB)
|     BIOS ROM     |
+------------------+  <- 0x000F0000 (960KB)
|  16-bit devices, |
|  expansion ROMs  |
+------------------+  <- 0x000C0000 (768KB)
|   VGA Display    |
+------------------+  <- 0x000A0000 (640KB)
|                  |
|    Low Memory    |
|                  |
+------------------+  <- 0x00000000
```

PC启动时是16bit的实模式，最大寻址空间是1MB。然后通过一系列的设置可以突破1MB的限制，进入32bit的保护模式，寻址空间达到4GB.


### The ROM BIOS
使用qemu调试，观察启动过程。打开两个终端窗口，在第一个终端中以debug的方式启动qemu （`make qemu-gdb` 或 `make-qemu-nox-gdb`），在第二个终端中使用gdb调试：
```bash
make gdb
gdb -n -x .gdbinit
GNU gdb (Ubuntu 9.2-0ubuntu1~20.04.1) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
.....
For help, type "help".
Type "apropos word" to search for commands related to "word".
+ target remote localhost:26000
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
warning: A handler for the OS ABI "GNU/Linux" is not built into this configuration
of GDB.  Attempting to continue with the default i8086 settings.

The target architecture is assumed to be i8086
[f000:fff0]    0xffff0: ljmp   $0xf000,$0xe05b
0x0000fff0 in ?? ()
+ symbol-file obj/kern/kernel
(gdb) 
```

这是执行的第一条汇编指令：
```s
[f000:fff0]    0xffff0: ljmp   $0xf000,$0xe05b
```
从这条指令中我们可以得到以下信息：
- PC 启动时开始执行的物理内存地址是 0x000f_fff0，这个地址对应的就是BIOS的位置
- PC 开始执行指令时对应的代码段 `CS=0xf000`, 指令指针 `IP=0xfff0`
- 第一条指令是一条 `jmp` 指令，它会跳转到 `CS=0xf000` ，`IP=0xe05b` 处

为什么是 `0xffff0`？这是一个固定的地址，x86架构就是这样设计的，具体参考[PC Booting](https://en.wikipedia.org/wiki/Booting)

**Exercise2:**
> 使用GDB的 `si`(Step Instruction)命令调试ROM BIOS, 了解BIOS的流程，可能用到的参考资料 [Phil Storrs I/O Ports Description](http://web.archive.org/web/20040404164813/members.iweb.net.au/~pstorr/pcbook/book2/book2.htm)

BIOS主要就是完成初始化硬件的工作，之后查询磁盘的启动分区，找到有效的启动分区，将其加载到内存中，最后将控制权移交给启动boot loader.


## Part 2: The Boot Loader
软盘或硬盘操作的最小单位是扇区（512byte）. 第一个扇区叫做启动扇区，bootloder代码就存放在512byte的空间中。如果是有效的启动扇区，BIOS就会将这512byte加载到0x7c00起始的物理内存地址，并跳转到此处执行 `CS=0x000:IP=0x7c00`. `0x7c00` ，又是一个看来奇怪又随意的地址，同样这也是一个固定的地址，具体参考[Why BIOS loads MBR into 0x7C00 in x86 ?](https://www.glamenv-septzen.net/en/view/6).


在6.828的lab中，bootloader的代码保护一个汇编文件(`boot/boot.S`)和一个c文件（`boot/main.c`）.Bootloader 主要完成以下两个功能：
1. 实现16bit实模式到32bit的保护模式切换，因为只有在保护模式下才能够访问1MB以外的内存空间。
2. bootloader通过I/O指令从磁盘中读取内核

在了解了bootloder的源码后，可以查看对应的反汇编的代码（`obj/boot/boot.asm`），这个反汇编的文件在每条汇编指令前又对应的物理地址。同样的，内核代码的反汇汇编文件（`obj/kern/kernel.asm`）也类似。


**Exercise3:**
> 调试需要使用GDB命令，可参考[ lab tools guide](https://pdos.csail.mit.edu/6.828/2018/labguide.html) . 调试 `boot/boot.S` 和 ` boot/main.c` 回答下面的问题。

- 处理器是从哪里开始执行32bit代码，具体是什么操作使得发生16bit到32bit的转换？
- bootloader的最后一条指令，以及它加载的kernel的第一条指令是什么？
- kernel的第一条指令在哪里（地址）？
- bootloader是怎么知道要从磁盘读取扇区的数量才能把整个kernel加载到内存？