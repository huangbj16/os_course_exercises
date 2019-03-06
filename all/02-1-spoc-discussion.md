# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习

   已提交，同时在piazza上提交了相关问题。

 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。

   已完成。

 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”

   从表面看，我观察发现Windows系统启动时会展示动态的加载画面和图标，而我使用的ubuntu系统启动时直接显示的是一行行的命令行输出信息。

   从内层看，80386开机启动时会经历从实模式到保护模式的切换，一是为了支持硬件查询等功能，二是要兼容之前的16位系统；而后者启动时则没有这样的一个步骤。

 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。

   答：？？？？？？

 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？

   中断：中断不是必须的，早期可以被轮询替代；它的作用是提高应用的运行效率，减少无效等待时间；它的不足是中断是异步的，导致每次中断触发系统都必须从用户态转到内核态才能实现外部设备读写，增加了耗费的时间，同时给程序的执行带来了不确定性。

   异常：是必须要有的。因为应用程序是不可信任的，因此应用程序在执行过程中出现的错误必须由操作系统经过检查，才能进行下一步操作。它的好处是保证了操作系统的鲁棒性，不会因为单个应用程序的错误而整体崩溃；它的不足是对于大部分情况下对错误无法给出具体的解决方案，只能采取kill程序的方式解决问题。

   系统调用：是必须要有的。因为应用程序是不可信任的，所以应用程序不能被允许直接使用特权指令，只能通过系统调用封装好的函数访问内核态的功能。它的好处是通过这种抽象的方式保证了系统调用的安全性和可靠性；它的不足是相对来说功能比较单一，需要在函数库层面再次封装才能满足应用程序的需求。

 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”

   | 向量号   | 名称         |
   | -------- | ------------ |
   | 0        | 除法错       |
   | 1        | 调试         |
   | 3        | 断点         |
   | 4        | 溢出         |
   | 5        | 界限检查     |
   | 6        | 无效操作码   |
   | 7        | 无80X87      |
   | 8        | 双重故障     |
   | 9        | NPX          |
   | 0AH      | 无效TSS      |
   | 0BH      | 段不存在     |
   | 0CH      | 堆栈段异常   |
   | 0DH      | 通用保护     |
   | 0EH      | 页异常       |
   | 10H      | 协处理器出错 |
   | 11H—0FFH | 软中断       |

 - 安装好ucore实验环境，能够编译运行lab8的answer

 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```

   此处列出了ucore中的系统调用：

   ```c++
   static int (*syscalls[])(uint32_t arg[]) = {
       [SYS_exit]              sys_exit,
       [SYS_fork]              sys_fork,
       [SYS_wait]              sys_wait,
       [SYS_exec]              sys_exec,
       [SYS_yield]             sys_yield,
       [SYS_kill]              sys_kill,
       [SYS_getpid]            sys_getpid,
       [SYS_putc]              sys_putc,
       [SYS_pgdir]             sys_pgdir,
       [SYS_gettime]           sys_gettime,
       [SYS_lab6_set_priority] sys_lab6_set_priority,
       [SYS_sleep]             sys_sleep,
       [SYS_open]              sys_open,
       [SYS_close]             sys_close,
       [SYS_read]              sys_read,
       [SYS_write]             sys_write,
       [SYS_seek]              sys_seek,
       [SYS_fstat]             sys_fstat,
       [SYS_fsync]             sys_fsync,
       [SYS_getcwd]            sys_getcwd,
       [SYS_getdirentry]       sys_getdirentry,
       [SYS_dup]               sys_dup,
   };
   ```

   相比之下Linux的系统调用更多更完整，功能更强大。

 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。

 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。

 - 在piazza上就lec3学习中不理解问题进行提问。

   已提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
- x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  第一个扇区读到的是主引导程序的信息，包括启动代码和磁盘分区的信息。之所以没有直接读入操作系统内核映像是因为操作系统的大小超过了一个扇区存储的限制，而BIOS的设计导致只能读一个扇区，所以需要间接读入的方法。另外BIOS不认识不同文件系统的形式。

- 比较UEFI和BIOS的区别。

  BIOS是一种历史较为悠久的开机启动程序，能完成开机硬件自检，系统自检，装入载入程序，引导操作系统的功能，它的缺点是为历史所限，大量服务非常冗余。

  UEFI是新型的BIOS，它在所有平台上提供统一的操作系统服务，并且更为安全，对引导程序的来源做了信任验证。

- 理解rcore中的Berkeley BootLoader (BBL)的功能。

  ```
  摘抄
  Since the size of BBL is larger than 64 KB (the size of the on-chip boot RAM), it is stored on an SD card and copied to DDR RAM during boot. An example program named ‘boot’ (`$TOP/fpga/board/$FPGA_BOARD/examples/boot.c`) is provided as the first stage bootloader. Please see [FPGA demo - Boot RISC-V Linux](https://www.lowrisc.org/docs/untether-v0.2/fpga-demo#boot) for more information.
  
  BBL runs after the soft reset as DDR is now mapped to the boot address `0x00000200`. The major function of the BBL is to initialize all peripherals, set up the page table and virtual memory, load the Linux kernel from SD to virtual memory, and finally boot the kernel.
  
  During the kernel execution, BBL continues running underneath the kernel as a hypervisor, serving all peripheral requests from Linux using the actual FPGA hardware.
  ```

  简而言之BBL是转为RISC-V设计的BootLoader。它需要有一个另一个BootLoader来负责从硬盘加载它。加载完成后，它的工作是初始化页表虚存，装载kernel和运行kernel。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

  0x55AA

- x86中在UEFI中的可信启动有什么作用？

  对引导进行验证，保证引导的来源是可靠的。

- RV中BBL的启动过程大致包括哪些内容？

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

  中断：中断不是必须的，早期可以被轮询替代；它的作用是提高应用的运行效率，减少无效等待时间；它的不足是中断是异步的，导致每次中断触发系统都必须从用户态转到内核态才能实现外部设备读写，增加了耗费的时间，同时给程序的执行带来了不确定性。

  异常：是必须要有的。因为应用程序是不可信任的，因此应用程序在执行过程中出现的错误必须由操作系统经过检查，才能进行下一步操作。它的好处是保证了操作系统的鲁棒性，不会因为单个应用程序的错误而整体崩溃；它的不足是对于大部分情况下对错误无法给出具体的解决方案，只能采取kill程序的方式解决问题。

  系统调用：是必须要有的。因为应用程序是不可信任的，所以应用程序不能被允许直接使用特权指令，只能通过系统调用封装好的函数访问内核态的功能。它的好处是通过这种抽象的方式保证了系统调用的安全性和可靠性；它的不足是相对来说功能比较单一，需要在函数库层面再次封装才能满足应用程序的需求。

- 中断、异常和系统调用的处理流程有什么异同？

  中断：异步

  异常：同步

  系统调用：二者皆可

  三者都通过中断向量表获得处理代码的起始位置跳转运行，不同的是系统调用还要再经过一层系统调用表。

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

  系统调用涉及到用户态和内核态的切换，同时需要切换堆栈；

  函数调用则是在用户态执行，堆栈也使用的是用户态的函数栈。

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
