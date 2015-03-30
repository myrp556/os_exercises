#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？

13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象


(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考如python代码或独自实现。
 - [页置换算法实现的参考实例](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab3/page-replacement-policy.py)
 >实现模拟lru算法
"""
 #include <iostream>
#include <stdio.h>
#include <cstring>
using namespace std;
int n, m;
struct Link{
	Link *next, *prev;
	int v;
	Link(int v){
		this->v=v;
		this->next=NULL;
		this->prev=NULL;
	}
};
Link *head;
void bindLink(Link *a, Link *b){
	if (a!=NULL) a->next=b;
	if (b!=NULL) b->prev=a;
}
void destroy(Link *a){
	if (a->prev!=NULL){
		(a->prev)->next=a->next;
	}else{
		if (a==head) head=a->next;
	}
	if (a->next!=NULL){
		(a->next)->prev=a->prev;
	}
}
int main(){
	freopen("lru.in", "r", stdin);
	freopen("lru.out", "w", stdout);
	scanf("%d%d", &n, &m);
	head=NULL;
	int len=0;
	for (int i=0; i<n; i++){
		int a;
		scanf("%d", &a);
		if (head==NULL){
			head=new Link(a);
			len++;
		}else{
			Link* v=head;
			while (v!=NULL){
				if (v->v == a || v->next==NULL) break;
				v=v->next;
			}
			Link *ptr=new Link(a);
			if (v->v == a || len>=m) destroy(v);
			else len++;
			bindLink(ptr, head);
			head=ptr;
		}
		for (Link *j=head; j!=NULL; j=j->next) printf("%d ", j->v);printf("\n");
	}
	fclose(stdin);
	fclose(stdout);
	return 0;
}
"""
>程序模拟从文件读入第一行n,m分别表示有n个输入指令以及m长度的物理块，之后n个数字表示模拟不同的调入内存请求。由于lru则是每次都会把调入的内容放在队首，如果长度超过了限制就会把最后的内容调出队列。最后每一次调入请求都会输出一次当前从头到尾的存储情况。
## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
