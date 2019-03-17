# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

## 与视频相关思考题

### 6.1	非连续内存分配的需求背景
 1. 为什么要设计非连续内存分配机制？

    解决连续内存分配时存在的问题，比如碎片的产生，分配颗粒度过大等等。


 1. 非连续内存分配中内存分块大小有哪些可能的选择？大小与大小是否可变?

    要求为2^n Byte，一般是512B，4KB等等。大小不可变。


 1. 为什么在大块时要设计大小可变，而在小块时要设计成固定大小？小块时的固定大小可以提供多种选择吗？

    小块时颗粒度已经足够细，可以满足各种程序的需求；如果还设计成可变大小的，反而提高了软硬件实现难度。

### 6.2	段式存储管理
 1. 什么是段、段基址和段内偏移？

    段是内存中连续的一段空间，用于存储一个进程所需的代码和数据。

    段基址和段内偏移一起构成了逻辑地址，通过转换基址将段基址转换为内存的起始地址，再通过段内偏移得到真正的物理地址。


 1. 段式存储管理机制的地址转换流程是什么？为什么在段式存储管理中，各段的存储位置可以不连续？这种做法有什么好处和麻烦？

    先查询段表获得段基址对应的内存的起始地址和长度，根据长度判断段内偏移是否越界。若越界则抛出异常；若不越界则将起始地址和段内偏移合并，得到物理地址，然后访问即可。

    因为段足够大，一般一个程序所需代码数据存储能被合理得分割成几个部分，如代码段，数据段，堆栈等。每个部分之间很少出现交叉访问的情况，因此不要求各段的存储位置连续。


### 6.3	页式存储管理
 1. 什么是页（page）、帧（frame）、页表（page table）、存储管理单元（MMU）、快表（TLB, Translation Lookaside Buffer）和高速缓存（cache）？

    页是虚拟内存中的一段固定长度的内存空间，帧是物理内存中页对应的内存空间。

    页表，MMU，TLB和Cache是实现虚实地址转换的软硬件机制。

 1. 页式存储管理机制的地址转换流程是什么？为什么在页式存储管理中，各页的存储位置可以不连续？这种做法有什么好处和麻烦？

    每当控制单元要求查询一个内存地址时，MMU首先将内存地址拆分成虚页号和页内偏移，然后查询快表。若TLB命中，则直接获得物理页号，就可以访问物理地址了；若TLB缺失，则需要先查询内存中的页表，获得物理页号，并更新TLB表项，然后根据获得的物理页号和页内偏移访问物理地址。

    因为有页表实现虚实地址转换，所以不需要连续存储，在进程看来它访问的虚拟地址仍然是连续的。


### 6.4	页表概述
 1. 每个页表项有些什么内容？有哪些标志位？它们起什么作用？

    表项有两部分内容，标志位和物理地址。

    标志位有存在位(resident bit)，修改位(dirty bit)，引用位(clock/reference bit)，只读位(read only OR read/write bit)。

    存在位表示物理内存中是否存放了这一页。

    修改位表示页面内容是否修改了。

    引用位说明在过去一段时间内是否访问过。

    只读位说明了这一页是否可读可写。

 1. 页表大小受哪些因素影响？


### 6.5	快表和多级页表
 1. 快表（TLB）与高速缓存（cache）有什么不同？

    快表中存储的是一部分频繁访问的虚拟页号和物理页号的对应关系。

    cache中存储的则是频繁访问的内存地址中存储的指令和数据。

 1. 为什么快表中查找物理地址的速度非常快？它是如何实现的？为什么它的的容量很小？

    因为它是用硬件实现在CPU上的，所以访问非常快；

    容量小则是因为空间和能耗的限制。

 1. 什么是多级页表？多级页表中的地址转换流程是什么？多级页表有什么好处和麻烦？

    多级页表是将虚拟页号分成多级，实现层次化查询。

    多级页表的地址转换是先将虚拟地址拆分成多级虚拟页号和页内偏移，再通过每一级的页号获得下一级的页表的起始地址，最终得到物理页号。

    好处是节省了许多无谓的页表存储空间。缺点是需要多次访问内存，费时费力。


### 6.6	反置页表
 1. 页寄存器机制的地址转换流程是什么？

    当查询虚拟地址时，先将虚拟地址hash得到在页寄存器中存放的下标，接着查看页寄存器中存储的是否是该虚拟地址对应的实际地址，是则查询成功，否则沿着链表继续查找。

 1. 反置页表机制的地址转换流程是什么？

    1. 当查询虚拟地址时，先将虚拟地址和进程号一起hash得到在页寄存器中存放的下标，接着查看页寄存器中存储的是否是该虚拟地址对应的实际地址以及进程号是否匹配，是则查询成功，否则沿着链表继续查找。

 1. 反置页表项有些什么内容？

    进程号，虚拟页号，物理页号。

### 6.7	段页式存储管理
 1. 段页式存储管理机制的地址转换流程是什么？这种做法有什么好处和麻烦？

    段号+页号+页内偏移。通过段号查询段表得到段内基址，进而得到页表基址，接着查询页号得到物理页号，接着通过物理页号和页内偏移即可访问物理地址。

    好处是可以很方便实现内存保护，内存共享和内存回收。

 1. 如何实现基于段式存储管理的内存共享？

    在两个需要共享内存的进程启动时，将它们重定向到同一个段上。

 1. 如何实现基于页式存储管理的内存共享？

    在页表中为二者存储相同的物理页号。

## 个人思考题
（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的

x86-64 CPU的实际虚拟地址大小为48位（64位中的低48位，高16位不用于地址转换），可寻址256TB地址空间。x86-64 CPU的实际物理地址大小，不同的CPU不相同，但最大不超过52位。

使用了分页机制之后，256TB的地址空 间被分成了固定大小的页（有三种大小，4KB，2MB，1GB），每一页或者被映射到物理内存，或者没有映射任何东西。

**x86-64使用的是多级页表的机制。**

CPU用来把虚拟地址转换成物理地址的信息存放在叫做"page map level 4"，" page directory pointer"，"page directory"（页目录），"page table"（页表）的结构里。每个进程都有自己的一套 PML4，PDPT，PD，PT结构。一个进程中的虚拟地址，最多需要四级转换，来得到对应的物理地址。

一 个虚拟地址，大小8个字节（64位，实际只使用低48位），包含着找到物理地址的信息，分为5个部分：第39位到第47位这9位（最高9位） 是"page map level 4"表中的索引，第30位到第38位这9位是"page directory pointer"表中的索引，第21位到第29位这9位是页目录中的索引，第12位到第20位这9位是页表中的索引，第0位到第11位 这12位（低12位）是页内偏移。

（引用：https://www.cnblogs.com/jiurl/p/4925007.html）

## 小组思考题
（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。

$$
150 * 10^{-9} * 90\% + x * 10\% = 0.5 * 10^{-6}\\
不在内存的页面访问时间为x = 3.65*10^{-6} = 3.65us
$$
（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
Virtual Address 6c74
  --> pde index:0x1b  pde contents:(valid 1, pfn 0x20)
    --> pte index:0x03  pte contents:(valid 1, pfn 0x61)
      --> Translates to Physical Address 0xc34 --> Value: 0x06

Virtual Address 6b22
  --> pde index:0x1a  pde contents:(valid 1, pfn 0x52)
    --> pte index:0x19  pte contents:(valid 1, pfn 0x47)
      --> Translates to Physical Address 0x8e2 --> Value: 0x1a

Virtual Address 03df
  --> pde index:0x00  pde contents:(valid 1, pfn 0x5a)
    --> pte index:0x1e  pte contents:(valid 1, pfn 0x05)
      --> Translates to Physical Address 0x0bf --> Value: 0x0f

Virtual Address 69dc
  --> pde index:0x1a  pde contents:(valid 1, pfn 0x52)
    --> pte index:0x0e  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

Virtual Address 317a
Virtual Address 4546
Virtual Address 2c03
Virtual Address 7fd7
Virtual Address 390e
Virtual Address 748b
```

比如答案可以如下表示： (注意：下面的结果是错的，你需要关注的是如何表示)
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，请说明原因。

（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python、ruby、C、C++、LISP、JavaScript等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，提交你的实现，并说明区别。

（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring/lecture06)
---

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。


## interactive　understand VM

[Virtual Memory with 256 Bytes of RAM](http://blog.robertelder.org/virtual-memory-with-256-bytes-of-ram/)：这是一个只有256字节内存的一个极小计算机系统。按作者的[特征描述](https://github.com/RobertElderSoftware/recc#what-can-this-project-do)，它具备如下的功能。

 - CPU的实现代码不多于500行；
 - 支持14条指令、进程切换、虚拟存储和中断；
 - 用C实现了一个小的操作系统微内核可以在这个CPU上正常运行；
 - 实现了一个ANSI C89编译器，可生成在该CPU上运行代码；
 - 该编译器支持链接功能；
 - 用C89, Python, Java, Javascript这4种语言实现了该CPU的模拟器；
 - 支持交叉编译；
 - 所有这些只依赖标准C库。

针对op-cpu的特征描述，请同学们通过代码阅读和执行对自己有兴趣的部分进行分析，给出你的分析结果和评价。
