个人思考题

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。

[x]大部分可以理解。不太理解的有int。

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？

[x]ucore中有些细节的地方不是特别的理解。还不清楚和普通x86的堆栈是否是一个格式。

哪些困难（请分优先级）会阻碍你自主完成lab实验？

[x]实验环境的配置>不能检测出错误>不能理解实验内容>错误理解实验>语法问题

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？

[x]根据代码基本的位置和执行过程中的顺序来判断，按照基本代码从上至下的执行顺序，以及遵循跳转规则，把映射的物理地址联系起来。

了解函数调用栈对lab实验有何帮助？

[x]理解函数调用栈会理解清楚函数运行的规则，也可以知道具体是如何运行函数的，有利于学习底层相关知识，也利于实验的进行和调试。

你希望从lab中学到什么知识？

[x]学习到操作系统本身的编写，执行规则和原理。同时能够认识到程序运行的底层基本知识，利于以后分析，处理各种的问题。


小组讨论题

搭建好实验环境，请描述碰到的困难和解决的过程。

[x]过程中发现一些地方不够明确。通过询问同学，并参照他们的配置修改才基本完成。

熟悉基本的git命令行操作命令，从github上 的 http://www.github.com/chyyuu/ucore_lab 下载 ucore lab实验

[x]
尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1

[x]
对于如下的代码段，请说明”：“后面的数字是什么含义

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

[x]这些就是分别区分了不同的位数，单位就是bit位。每一位就是表示占据了多少位的空间的字段，代表了不同的数字位。


对于如下的代码段，

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
如果在其他代码段中有如下语句，

unsigned intr;
intr=8;
SETGATE(intr, 0,1,2,3);
请问执行上述指令后， intr的值是多少？

[x]65538


请分析 list.h内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序

[x] 
list.h中定义了双链表的操作。可以像链表一样使用，初始化一个链表头开始向链表尾添加元素。并且可以得到一个链表元素前后的元素。

#include <iostream>
#include "list.h"
using namespace std;

int main()
{
     list_entry_t* a = new list_entry_t;
     list_entry_t* b = new list_entry_t;
     list_entry_t* c = new list_entry_t;
     list_init(a);
     list_add(a, b);
     list_add(b, c);
     cout << list_prev(b) << ' ' << a;
     system("pause");
     return 0;   
}
开放思考题

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。

[x]
