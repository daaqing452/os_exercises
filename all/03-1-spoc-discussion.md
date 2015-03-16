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

> * 最优匹配  优点：大部分分配尺寸较小，效果较好；缺点：外部随便较小无法利用，释放分区较慢
> * 最差匹配：优点：中等大小内存分配效果好，避免太多小碎片；缺点：破坏大空闲分区，释放分区较慢
> * 最先匹配：优点：简单、高地址有较大空闲分区；缺点：外部碎片较多，分配大分区较慢
> * buddy systemm：优点：选取和释放都很灵活，分区利用率较高；缺点：大分区内部碎片较大
> * 新算法：随机分配算法——每次随机找一个比需求大的空闲快分配


## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

```
#include <cstdio>

//	页
struct Page {
	int _;
};

//	内存
Page memory[1 << 20];

//	一段存储单元
struct Unit {
	Page* start;
	int size;
	Unit *prev, *next;
};

//	内存管理
struct Manager {
	
	//	未使用内存链表
	Unit *unused;
	
	//	已使用内存链表
	Unit *used;
	
	Manager() {
		Unit *a = new Unit();
		a->start = memory;
		a->size = 1 << 20;
		a->prev = a->next = NULL;
		unused = a;
		used = NULL;
	}
	
	//	链表中删除
	void deleteUnit(Unit *a, Unit *&head) {
		if (a->prev != NULL) a->prev->next = a->next;
		if (a->next != NULL) a->next->prev = a->prev;
		if (head == a) head = a->next;
		delete a;
	}
	
	//	链表中将新单元与前后建立连接
	void renewLink(Unit *a) {
		if (a->prev != NULL) a->prev->next = a;
		if (a->next != NULL) a->next->prev = a;
	}
	
	//	链表中前后合并
	void mergeUnit(Unit *a) {
		Unit *x = a->prev;
		if (x != NULL && x->start + x->size == a->start) {
			x->size += a->size;
			deleteUnit(a, unused);
			a = x;
		}
		Unit *y = a->next;
		if (y != NULL && a->start + a->size == y->start) {
			a->size += y->size;
			deleteUnit(y, unused);
		}
	}
	
	//	申请内存
	Page* alloc(int s) {
	
		//	归整为4的倍数
		s = (s + 3) / 4;
		
		//	从前往后找空闲内存
		Unit *a = unused;
		Page *result = NULL;
		while (a != NULL) {
			if (a->size < s) {
				a = a->next;
				continue;
			}
			
			//	找到足够大的内存
			Unit *b = new Unit();
			b->start = result = a->start;
			b->size = s;
			b->prev = NULL;
			b->next = used;
			renewLink(b);
			used = b;
			
			//	处理剩下的内存
			if (a->size > s) {
				a->start = a->start + s;
				a->size -= s;
			} else {
				deleteUnit(a, unused);
			}
			break;
		}
		
		return result;
	}
	
	//	释放内存
	void free(Page *page) {
		
		//	找到指定的单元
		Unit *a = used;
		int s = 0;
		while (a != NULL) {
			if (a->start == page) break;
			a = a->next;
		}
		s = a->size;
		deleteUnit(a, used);
		
		a = unused;
		if (a == NULL || a->start > page) {
			//	新空闲插在最前面
			Unit *b = new Unit();
			b->start = page;
			b->size = s;
			b->prev = NULL;
			b->next = unused;
			renewLink(b);
			b->next->prev = b;
			unused = b;
			mergeUnit(b);
		} else {
			//	新空闲插在中间
			while (a != NULL) {
				if (a->next == NULL || a->next->start > page) break;
				a = a->next;
			}
			Unit *b = new Unit();
			b->start = page;
			b->size = s;
			b->prev = a;
			b->next = a->next;
			renewLink(b);
			mergeUnit(b);
		}
	}

	void print() {
		Unit *a = unused;
		printf("unused\t");
		while (a != NULL) {
			printf(" -> (%d,%d)", a->start - memory, a->size);
			a = a->next;
		}
		a = used;
		printf("\nused\t");
		while (a != NULL) {
			printf(" -> (%d,%d)", a->start - memory, a->size);
			a = a->next;
		}
		printf("\n\n");
	}
};


int main() {
	
	Manager *manage = new Manager();
	manage->print();
	manage->alloc(100 * 4);
	manage->print();
	manage->alloc(1000 * 4);
	manage->print();
	manage->alloc(10000 * 4);
	manage->print();
	manage->free(memory + 0);
	manage->print();
	manage->free(memory + 1100);
	manage->print();
	manage->free(memory + 100);
	manage->print();
	
	return 0;
}
```

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



