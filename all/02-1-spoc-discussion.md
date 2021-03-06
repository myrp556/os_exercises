#lec 3 SPOC Discussion

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
 1. 比较UEFI和BIOS的区别。
 1. 描述PXE的大致启动流程。

## 3.2 系统启动流程
 1. 了解NTLDR的启动流程。
 1. 了解GRUB的启动流程。
 1. 比较NTLDR和GRUB的功能有差异。
 1. 了解u-boot的功能。

## 3.3 中断、异常和系统调用比较
 1. 举例说明Linux中有哪些中断，哪些异常？
 1. Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)

>Linux的系统调用有200多条。调用有 进程控制，文件系统控制，系统控制，存管管理，网络管理，socket控制，用户管理，进程间通信

```
  + 采分点：说明了Linux的大致数量（上百个），说明了Linux系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
 1. 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)
 
>ucore lab8指令数量有22个。大致功能分类有进程控制，系统控制，文件系统控制。
 ```
  + 采分点：说明了ucore的大致数量（二十几个），说明了ucore系统调用的主要分类（文件操作，进程管理，内存管理等）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
## 3.4 linux系统调用分析
 1. 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 >objdump将.exe文件反汇编。nm可以列出文件符号清单，file可以列出文件的属性和类型。这里显示了这是一个可执行的文件。
.include "defines.h"
.data
hello:
	.string "hello world\n"

.globl	main
main:
	movl	$SYS_write,%eax
	movl	$STDOUT,%ebx
	movl	$hello,%ecx
	movl	$12,%edx
	int	$0x80

	ret
	代码中.data指定了数据段。其中定义了hello是一个"hello world"的字符串。之后定义全局的符号main。之后再寄存器eax上指定系统输出指令，ebx上指定到标准屏幕输出，ecx上指定需要输出的字符串是之前定义的数据段hello，edx上指定了输出长度是12，最后intx80退出结束，最后ret返回函数。

 ```
  + 采分点：说明了objdump，nm，file的大致用途，说明了系统调用的具体含义
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 
 ```
 
 1. 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 >strace可以跟踪一个进程的系统调用，返回细节的运行时间，参数，返回值等。
cpu加电，完成初始化，读取第一条指令。cpu初始为16位的实模式，之后bios开始初始化，硬件开始自检，检测内存，硬盘等关键部分是否处在工作状态。查找执行接口的bios，进行设备初始化。之后执行系统bios，系统开始自检。检测完成后从指定的磁盘或者设备中读入第一块扇区。之后必须跳转到对应的引导分区中。有了这些记录才能跳转到合适的活动分区中，之后根据平台，硬盘执行跳转指令，目标则是启动代码，跳转到加载程序。以上完成加载程序，之后执行加载程序，根据系统也会不同。 选择启动内核和参数。之后根据配置信息，加载内核并交给内核处理。

 ```
  + 采分点：说明了strace的大致用途，说明了系统调用的具体执行过程（包括应用，CPU硬件，操作系统的执行过程）
  - 答案没有涉及上述两个要点；（0分）
  - 答案对上述两个要点中的某一个要点进行了正确阐述（1分）
  - 答案对上述两个要点进行了正确阐述（2分）
  - 答案除了对上述两个要点都进行了正确阐述外，还进行了扩展和更丰富的说明（3分）
 ```
 
## 3.5 ucore系统调用分析
 1. ucore的系统调用中参数传递代码分析。
 1. ucore的系统调用中返回结果的传递代码分析。
 1. 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
 1. 以ucore lab8的answer为例，尝试修改并运行代码，分析ucore应用的系统调用执行过程。
 
## 3.6 请分析函数调用和系统调用的区别
 1. 请从代码编写和执行过程来说明。
   1. 说明`int`、`iret`、`call`和`ret`的指令准确功能
 
