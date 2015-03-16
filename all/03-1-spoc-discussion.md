# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

> 最优匹配：


>  buddy systemm分配算法：尽管伙伴内存算法在内存碎片问题上已经做的相当出色，但是该算法中，一个很小的块往往会阻碍一个大块的
合并，一个系统中，对内存块的分配，大小是随机的，一片内存中仅一个小的内存块没有释放，旁边两个大的就不能合并。算法中有一定的
浪费现象，伙伴算法是按2的幂次方大小进行分配内存块，当然这样做是有原因的，即为了避免把大的内存块拆的太碎，更重要的是使分配
和释放过程迅速。但是他也带来了不利的一面，如果所需内存大小不是2的幂次方，就会有部分页面浪费。有时还很严重。比如原来是1024个
块，申请了16个块，再申请600个块就申请不到了，因为已经被分割了。另外拆分和合并涉及到 较多的链表和位图操作，开销还是比较大的。

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

```
如何表示空闲块？ 如何表示空闲块列表？ 
[(start0, size0),(start1,size1)...]
在一次malloc后，如果根据某种顺序查找符合malloc要求的空闲块？如何把一个空闲块改变成另外一个空闲块，或消除这个空闲块？如何更新空闲块列表？
在一次free后，如何把已使用块转变成空闲块，并按照某种顺序（起始地址，块大小）插入到空闲块列表中？考虑需要合并相邻空闲块，形成更大的空闲块？
如果考虑地址对齐（比如按照4字节对齐），应该如何设计？
如果考虑空闲/使用块列表组织中有部分元数据，比如表示链接信息，如何给malloc返回有效可用的空闲块地址而不破坏
元数据信息？
伙伴分配器的一个极简实现
http://coolshell.cn/tag/buddy
```

#include <iostream>
#include <stdio.h>
#include <stdlib.h>
using namespace std;
int base = 0;
int size = 2147483647;
struct addr_list
{
	int begin;
	int length;
	bool is_free;
	addr_list *next;
};
addr_list* fr;
void init()
{
	fr = new addr_list();
	fr->begin = base;
	fr->length = size;
	fr->is_free = true;
	fr->next = NULL;
	return;
}
void add_malloc(int len)
{
	int ret_begin = 0;
	int ret_length = 0;
	if(fr == NULL)
	{
		printf("there is no free addr\n");
		return;
	}
	addr_list* p = fr;
	addr_list* q = p->next;
	addr_list* ret = fr;
	while(p != NULL)
	{
		if(q != NULL)
		{
			if(ret->length < q->length)
				ret = q;
			p = q;
			q = p->next;
		}
		else
			break;
	}
	ret_begin = ret->begin;
	if(ret->length < len)
	{
		printf("can not find this addr\n");
		return;
	}
	int a = 1; 
	for(int i=0; ; i++)
	{
		if(a >= len)
			break;
		a = a * 2;
	}
	ret_length = a;
	addr_list* b = fr;
	ret->begin = ret->begin + ret_length;
	ret->length = ret->length - ret_length;
	if(ret->length == 0)
	{
		addr_list* b = fr;
		addr_list* c = b->next;
		if(ret == fr)
			fr = fr->next;
		while(c != NULL)
		{
			if( c == ret)
			{
				b->next = ret->next;
			}
			else
			{
				b = c;
				c = b->next;
			}
		}
		ret = NULL;
	}
	printf("the molloc addr's begin addr is %d, length is %d\n", ret_begin, ret_length);
	return;
}
void addr_back(int begin, int length)
{
	addr_list* a = new addr_list;
	a->begin = begin;
	a->length = length;
	a->is_free = true;
	a->next = NULL;
	if(fr == NULL){
		fr = a;
		return;
	}
	else{
		if(begin < fr->begin){
			if(begin + length == fr->begin){
				fr->begin = begin;
				fr->length = fr->begin + length;
			}
			return;
		}	
		addr_list* p = fr;
		addr_list* q = fr->next;
		while(p != NULL){
			if(q != NULL){
				if(p->begin < begin && q->begin >begin){
					p->next = a;
					a->next = q;
					if(p->begin + p->length == begin){
						p->length = p->length + length;
						p->next = q;
						if(p->begin + p->length == q->begin){
							p->length = p->length + q->length;
							p->next = q->next;
						}
					}
					else{
						if(begin + length == q->begin){
							a->length = a->length + length;
							a->next = q->next;
						}
					}
					break;
				}
			}
			else{
				if(p->begin + p->length == begin){
					p->length = p->length + length;
					p->next = NULL;
				}
				else
					p->next = a;
				break;
			}
			p = q;
			q = p->next;
		}
	}
}



--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



