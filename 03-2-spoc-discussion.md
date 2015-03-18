# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
```
  + 采分点：说明64bit CPU架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了64bit CPU支持的物理内存大小限制（1分）
  - 正确描述了64bit CPU下的多级页表的级数和多级页表的结构或反置页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表或反置页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- [x]  

>  64位机器最多支持256TB内存。

> 对于32位的机器，采用两级页表结构是合适的；但对于64位的机器，如果页面大小仍采用4 KB即212 B，
那么还剩下52位，假定仍按物理块的大小(212位)来划分页表，则将余下的42位用于外层页号。此时在外
层页表中可能有4096 G个页表项，要占用16384 GB的连续内存空间。必须采用多级页表，将外层页表再
进行分页，也是将各分页离散地装入到不相邻接的物理块中，再利用第2级的外层页表来映射它们之间
的关系。　对于64位的计算机，如果要求它能支持2６４(=1844744 TB)规模的物理存储空间，则即使是
采用三级页表结构也是难以办到的；而在当前的实际应用中也无此必要。

> 在分页模式下, 操作系统可以创建一个为所有任务公用的4GB虚拟内存空间, 也可以为每一个任务创建
独立的4GB虚拟内存空间, 这都是可行的. 当一个程序加载时, 操作系统既要在左边的虚拟内存中分配段空
间, 又要在右边的物理内存中分配相应的页面. 因此, 第一步骤是寻找空闲的段空间, 该段空间既没有被
其他程序使用, 也没有被同一程序内的其他段使用. 比如上图, 假设已经成功找到并分配了一个段空间, 
基地址为0x00200000, 长度为8200字节.
页的最小尺寸是4KB, 也就是4096字节, 因此, 8200字节的段, 需要占用3个页面, 其中最后一个页面只用
了8个字节, 其余都是浪费着, 但这无关紧要, 如果允许页共享, 多个段或多个程序可以用同一个页来存
放各自的数据. 在分段之后, 操作系统的任务就是把段拆开, 并分别映射到物理页. 注意, 段必须是连续
的, 但不要求所分配的页都是连续的, 挨在一起的.

## 小组思考题
---

（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。 

- [x]  

> 500=0.9\*150+0.1\*x

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
Virtual Address 6b22
Virtual Address 03df
Virtual Address 69dc
Virtual Address 317a
Virtual Address 4546
Virtual Address 2c03
Virtual Address 7fd7
Virtual Address 390e
Virtual Address 748b
```

比如答案可以如下表示：
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

> Virtual Address 6c74 11011 00011 10100 
  --> pde index : 0x1b pde contents:(valid 1,pfn 0x20)
    --> pte index : 0x03 pte contents :(valid 1 pfn 0x61)
      --> Translates to Physical Address 0x6114 --> Value: 06
Virtual Address 6b22 11010 11001 00010
  --> pde index : 0x1a pde contents:(valid 1,pfn 0x52)
    --> pte index : 0x19 pte contents :(valid 1,pfn 0x47)
      --> Translates to Physical Address 0x4702 --> Value: 1a
Virtual Address 03df
  --> pde index : 0x00 pde contents:(valid 1,pfn 0x5a)
    --> pte index : 0x1e pte contents :(valid 1 pfn 0x05)
      --> Translates to Physical Address  --> Value: f
结果：
Virtual Address 69dc
  -->Fault (page table entry not valid)
Virtual Address 317a
  --> 0x1e
Virtual Address 4546
  -->Fault (page table entry not valid)
Virtual Address 2c03
  -->0x16
Virtual Address 7fd7
  -->Fault (page table entry not valid)
Virtual Address 390e
  -->Fault (page directory entry not valid)
Virtual Address 748b
  -->Fault (page table entry not valid)


（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。


（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

> 利用反置页表进行地址变换时，是用进程标志符和页号去检索反置页表；若检索完整个页表都未找到与之匹配的页表项，
表明此页此时尚未调入内存，对于具有请求调页功能的存储器系统应产生请求调页中断，若无此功能则表示地址出错；如
果检索到与之匹配的表项，则该表项的序号i便是该页所在的物理块号，将该块号与页内地址一起构成物理地址。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2015/lecture06#head-1f58ea81c046bd27b196ea2c366d0a2063b304ab)
--- 

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。

--- 
