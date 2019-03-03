# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  硬件设计上要支持虚实地址转换，有足够大的内存和硬盘空间。

  特权指令应该包括切换进程，虚实地址转换中的替换，系统调用。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  实模式是一种古老的OS模式，是X86-32为兼容DOS而提出的模式，物理地址空间不超过1MB；保护模式则完整地使用了32位的地址空间，物理内存大小为4GB。

  物理地址是处理器向物理内存存取数据时使用的地址；

  线性地址是应用程序在虚存中使用的地址，在无页机制时与物理地址相同，有页机制时经过页机制转换为物理地址；

  逻辑地址是应用程序直接使用的地址。

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

":"后面的数字代表了地址切分后的值，比如gd_off_15_0代表offset的低16位。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

intr的值是3。

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

   我阅读了Lab1的bootasm.S， 它实现了bootloader的功能，既能进入实模式，也能通过bootstrap转换成保护模式。同时，在进入两个模式的过程中，模式特有的寄存器都完成了初始化。

   ```assembly
   #切换模式
   
   seta20.1:
       inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
       testb $0x2, %al
       jnz seta20.1
   
       movb $0xd1, %al                                 # 0xd1 -> port 0x64
       outb %al, $0x64                                 # 0xd1 means: write data to 8042's P2 port
   
   seta20.2:
       inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
       testb $0x2, %al
       jnz seta20.2
   
       movb $0xdf, %al                                 # 0xdf -> port 0x60
       outb %al, $0x60                                 # 0xdf = 11011111, means set P2's A20 bit(the 1 bit) to 1
   
       # Switch from real to protected mode, using a bootstrap GDT
       # and segment translation that makes virtual addresses
       # identical to physical addresses, so that the
       # effective memory map does not change during the switch.
       lgdt gdtdesc
       movl %cr0, %eax
       orl $CR0_PE_ON, %eax
       movl %eax, %cr0
   
       # Jump to next instruction, but in 32-bit code segment.
       # Switches processor into 32-bit mode.
       ljmp $PROT_MODE_CSEG, $protcseg
   
   #寄存器初始化
   
   xorw %ax, %ax                                   # Segment number zero
   movw %ax, %ds                                   # -> Data Segment
   movw %ax, %es                                   # -> Extra Segment
   movw %ax, %ss                                   # -> Stack Segment
   
   movw $PROT_MODE_DSEG, %ax                       # Our data segment selector
   movw %ax, %ds                                   # -> DS: Data Segment
   movw %ax, %es                                   # -> ES: Extra Segment
   movw %ax, %fs                                   # -> FS
   movw %ax, %gs                                   # -> GS
   movw %ax, %ss                                   # -> SS: Stack Segment
   ```

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
