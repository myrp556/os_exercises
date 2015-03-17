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

>  最优匹配：优势：由于每次都是找到最小的一个合适的空间使用，因此利用量较高，这样碎片会少一些。 劣势：由于多次利用可能也会使得大量的小碎片变的不能利用，同时由于需要排序当前的空间，也需要代价。同时在找到相邻可以合并或者释放一个区域的时候处理会更麻烦。
>最差匹配：优势：由于每次都会找到最大的空间使用，因此产生的外部碎片也会很有可能会被之后继续利用。同样产生小碎片的可能性也会降低一些。 
劣势：同样因为排序空间需要额外的处理，因为每次都会分配最大的空间，这样可能会造成更多的碎片或者之后需要大量空间的进程就不能找到合适的资源。
>最先匹配；优势：这种做法比较简单，易于实现。相对维护也很容易。 劣势：由于只是单一的找到最先符合的空间使用，如果有，由于分配原则，可能会使得地位地址有大量的碎片，而高位地址又没办法得到合理的利用，使得整体的速度变慢。
>buddy system：优势：可以详细的存储使用信息，同时速度也比较优秀。空间利用率高。劣势：实现复杂，过程繁琐。

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
```
#include <iostream>
#include <stdio.h>
#include <algorithm>
using namespace std;

#define BLOCKS 100000
#define MAP 65550
#define INF 23333333

struct Link{
	Link *next, *prev;
	int num;
	Link(int num=0, Link* next=NULL, Link* prev=NULL){
		this->num=num;
		this->next=next;
		this->prev=next;
	}
	bool head(){
		return this->prev==NULL;
	}
};
struct tBlock{
	int l, r, len, no;
	bool flag;
	Link *tar;
	tBlock(int l=0, int r=0, Link* tar=NULL){
		this->l=l;
		this->r=r;
		this->tar=tar;
		this->flag=true;
		this->len=r-l;
	}
	void disable(){
		this->flag=false;
	}
	bool take(int no){
		if (!this->flag) return false;
		this->flag=false;
		this->no=no;
		return true;
	}
	bool is(){
		return this->flag;
	}
	tBlock& operator=(const tBlock& t){
		this->l=t.l;this->r=t.r;
		this->len=t.len;
		this->flag=t.flag;
		this->tar=t.tar;
		return *this;
	}
};

struct Task{
	int l, r;
	Task(int l=0, int r=0){
		this->l=l;
		this->r=r;
	}
	Task& operator=(const Task& t){
		this->l=t.l;
		this->r=t.r;
		return *this;
	}
};

const bool operator<(const tBlock &a, const tBlock &b){
	if (!a.flag) return false;
	if (!b.flag) return true;
	if (a.len != b.len) return a.len<b.len;
	return a.l<b.l;
}
const bool operator==(const tBlock &a, const tBlock &b){
	if (!a.flag || !b.flag) return false;
	return a.len == b.len;
}
const bool operator>(const tBlock &a, const tBlock &b){
	if (a==b) return false;
	return !(a<b);
}
int n, m, tblock;
tBlock block[BLOCKS];
Link* link_head;
int map[MAP], busy[MAP];
Task task[MAP];

void bind_link(Link* &l, Link* &r){
	l->next=r;r->prev=l;
}

bool add(int no, int len){
	int del=0;
	for (int i=0; i<tblock; i++){
		if (block[i].flag && block[i].len>=len){
			int ti=i;
			del=-1;
			Link *tlink=block[i].tar;
			Link *next=tlink->next;
			Link *prev=tlink->prev;
			Link *clink=next;
			block[ti].flag=false;
			task[no]=Task(block[ti].l, block[ti].r);
			if (block[i].len>len){
				Link *nlink=new Link(tblock);
				ti=tblock;
				block[tblock++]=tBlock(block[i].l+len, block[i].r, nlink);
				task[no].r=block[i].l+len;
				if (next != NULL) bind_link(nlink, next);
				clink=nlink;
			}
			if (prev != NULL && clink != NULL) bind_link(prev, clink);
			if (prev == NULL && clink != NULL) link_head=clink;
			
			printf("take %d %d\n", task[no].l, task[no].r);
			sort(block, block+tblock);
			tblock+=del;
			busy[no]=true;
			return true;
		}
	}
	return false;
}

bool loose(int no){
	busy[no]=false;
	int l=task[no].l, r=task[no].r;
	printf("release %d %d\n", l, r);
	int del=0;
	int minl=INF, minr=INF;
	int tl=-1, tr=-1;
	bool get=true;
	while (get){
		tl=-1;tr=-1;
		get=false;
		for (int i=0; i<tblock; i++){
			if (block[i].r<=l && l-block[i].r<minl){
				minl=l-block[i].r;
				tl=i;
			}
			if (r<=block[i].l && block[i].l-r<minr){
				minr=block[i].l-r;
				tr=i;
			}
		}
		
		if (tl>=0 && block[tl].r==l){
			block[tl].disable();
			l=block[tl].l;
			del--;
			get=true;
		}
		if (tr>=0 && r==block[tr].l){
			block[tr].disable();
			r=block[tr].r;
			del--;
			get=true;
		}
	}
	Link* nlink=new Link(tblock);
	if (tl>=0 && block[tl].tar != NULL) bind_link(block[tl].tar, nlink);
	else link_head=nlink;
	if (tr>=0 && block[tr].tar != NULL) bind_link(nlink, block[tr].tar);
	block[tblock++]=tBlock(l, r, nlink);
	sort(block, block+tblock);
	tblock+=del;
	return true;
}


int main(){
	freopen("mem.in", "r", stdin);
	freopen("mem.out", "w", stdout);
	scanf("%d", &m);
	tblock=0;
	link_head=new Link(tblock, NULL, NULL);
	block[tblock++]=tBlock(0, m, link_head);
	
	char op[10];
	int t;
	
	while (scanf("%s", &op)!=EOF){
		int len, no;
		if (strcmp(op, "a")==0){
			scanf("%d%d", &no, &len);
			if (busy[no]){
				printf("No. %d task is already running.\n", no);
				continue;
			}
			
			if (!add(no, len)){
				printf("Not enough space for No. %d task.\n", no);
				continue;
			};
			printf("%d space has been Allocated for task %d.\n", len, no);
		}else{
			scanf("%d", &no);
			if (!busy[no]){
				printf("No. %d task is not running.\n", no);
				continue;
			}
			
			loose(no);
			printf("No. %d task has been released.\n", no);
		}
		for (int i=0; i<tblock; i++) printf("%d %d %d %d\n", block[i].l, block[i].r, block[i].len, block[i].flag);
	}
	
	fclose(stdin);
	fclose(stdout);
	return 0;
}
```
```
输入测试
100
a 1 10
a 2 15
a 3 10
r 2
a 4 5
a 5 20
a 6 10
r 6
a 7 20
r 5
```
```
输出
take 0 10
10 space has been Allocated for task 1.
10 100 90 1
take 10 25
15 space has been Allocated for task 2.
25 100 75 1
take 25 35
10 space has been Allocated for task 3.
35 100 65 1
release 10 25
No. 2 task has been released.
10 25 15 1
35 100 65 1
take 10 15
5 space has been Allocated for task 4.
15 25 10 1
35 100 65 1
take 35 55
20 space has been Allocated for task 5.
15 25 10 1
55 100 45 1
take 15 25
10 space has been Allocated for task 6.
55 100 45 1
release 15 25
No. 6 task has been released.
15 25 10 1
55 100 45 1
take 55 75
20 space has been Allocated for task 7.
15 25 10 1
75 100 25 1
release 35 55
No. 5 task has been released.
15 25 10 1
35 55 20 1
75 100 25 1

```
>将所有空闲的内存块用链表块表示，地址之间是相邻的。链表Link类表示的就是空闲的地址，tar表示的区间信息，next是在它之后的地址块，prev是之前的地址块。对所有的区间信息排序，每次从最小的中找到第一个合适的地址为分配请求分配一块区域，如果更大就把高位的地址分开作为新的一块和之前的块地址链接在一起。如果有释放请求，可以知道这个编号的进程占用的地址，并重新把这一块地址加入到链表合适的地方，同时合并两边连续的块。
>测试中第一个数表示总共的地址大小。然后多行，每行第一个字母如果是a就是一个分配地址请求，跟上t c，表示编号t的进程要求分配c大小的地址，以及第一个字母r表示一个释放空间请求，跟上一个数字t表示释放编号为t的进程。
>输出多行，对每一个请求都输出一个结果，如果是分配请求会输出具体为它分配的哪一块地址的空间，同时每次请求后会输出所有当前空闲的空间块。
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


