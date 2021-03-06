# 同步互斥(lec 17) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 背景
 - 请给出程序正确性的定义或解释。
 - 在一个新运行环境中程序行为与原来的预期不一致，是错误吗？
 - 程序并发执行有什么好处和障碍？
 - 什么是原子操作？

### 现实生活中的同步问题

 - 家庭采购中的同步问题与操作系统中进程同步有什么区别？
 - 如何通过枚举和分类方法检查同步算法的正确性？
 - 尝试描述方案四的正确性。
 - 互斥、死锁和饥饿的定义是什么？

### 临界区和禁用硬件中断同步方法

 - 什么是临界区？
 - 临界区的访问规则是什么？
 - 禁用中断是如何实现对临界区的访问控制的？有什么优缺点？

### 基于软件的同步方法

 - 尝试通过枚举和分类方法检查Peterson算法的正确性。
 - 尝试准确描述Eisenberg同步算法，并通过枚举和分类方法检查其正确性。

### 高级抽象的同步方法

 - 如何证明TS指令和交换指令的等价性？
 - 为什么硬件原子操作指令能简化同步算法的实现？
 
## 小组思考题

1. （spoc）阅读[简化x86计算机模拟器的使用说明](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/lab7-spoc-exercise.md)，理解基于简化x86计算机的汇编代码。

2. （spoc)了解race condition. 进入[race-condition代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/race-condition)。

 - 执行 `./x86.py -p loop.s -t 1 -i 100 -R dx`， 请问`dx`的值是什么？
 ```
 程序只有一个线程0
 dx=-1
 ```
 - 执行 `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx` ， 请问`dx`的值是什么？
 ```
 程序分开成两个线程开始分别运行，线程0结束后才运行线程1
 dx=-1
 ```
 - 执行 `./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx`， 请问`dx`的值是什么？
 ```
 线程分成两个线程执行。
 dx=-1
 ```
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 1 -M 2000`, 请问变量x的值是什么？
 ```
 程序只有一个线程0，变量x是1
 ```
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000`, 请问变量x的值是什么？为何每个线程要循环3次？
 ```
 程序分开成两个线程开始分别运行，线程0结束后才运行线程1，线程0结束后线程1开始在x上同样开始加值。
 x=6
 每个线程循环3次为了使每个线程执行足够的指令。
 ```
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0`， 请问变量x的值是什么？
 ```
 程序分成两个线程执行并相互切换。
 x=2
 在中途对x变量读取或者写入出现了冲突
 ```
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 1`， 请问变量x的值是什么？
 ```
 情况同上
 x=1
 ```
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 2`， 请问变量x的值是什么？ 
 ```
 情况同上
 x=1
 ```
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1`， 请问变量x的值是什么？ 
 ```
 每个线程都是执行一条指令就切换。
 x=1
 ```

3. （spoc） 了解software-based lock, hardware-based lock, [software-hardware-lock代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/software-hardware-locks)

  - 理解flag.s,peterson.s,test-and-set.s,ticket.s,test-and-test-and-set.s 请通过x86.py分析这些代码是否实现了锁机制？请给出你的实验过程和结论说明。能否设计新的硬件原子操作指令Compare-And-Swap,Fetch-And-Add？
```
Compare-And-Swap

int CompareAndSwap(int *ptr, int expected, int new) {
  int actual = *ptr;
  if (actual == expected)
    *ptr = new;
  return actual;
}
```

```
Fetch-And-Add

int FetchAndAdd(int *ptr) {
  int old = *ptr;
  *ptr = old + 1;
  return old;
}
```
```
flag.s:
	设置i=1,2出现反例
	执行错误
perterson:
	设置i=1,2出现反例
	执行错误
test-and-set.s, test-and-test-and-set:
	指令1 python x86.py -p test-and-set.s -a bx=2 -t 2 -i 2 -R ax -c
		执行2个线程，设置循环执行2次，切换频率为2，显示ax寄存器的内容
	指令2 python x86.py -p test-and-set.s -a bx=2 -t 2 -i 1 -R ax -c
		执行2个线程，设置循环执行2次，切换频率为1，显示ax寄存器的内容
	在线程0首先执行到aquire的时候检查发现没有上锁，因此可以进入下一步执行，
	另外一个线程则因为metex已经被赋值而不能进入临界区，继续等待线程0执行完毕。
	等到线程0完成一次临界区访问后解锁，切换到线程1就可以让线程1进入临界区执行。
	从执行的结果中看到，可以完成互斥锁的功能。
ticket.s:
	参数-t 3 -i 1
	执行结果正确
```


