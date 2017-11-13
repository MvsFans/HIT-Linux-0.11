# 基于内核栈切换的进程切换

1. [实验内容](#实验内容)
2. [实验过程](#实验过程)
  1. [实验结果](#实验结果)
  2. [实验结果](#实验分析)
    1. [为什么要从基于TSS的任务切换改为基于内核栈切换的任务切换](#为什么要从基于TSS的任务切换改为基于内核栈的任务切换)
	2. [是否还需要任务状态段TSS](#是否还需要任务状态段TSS)

## 实验内容

## 实验过程

### 实验结果

### 实验分析

#### 1. 为什么要从基于TSS的任务切换改为基于内核栈切换的任务切换
Linux 0.11利用80x86硬件提供的机制：通过执行`ljmp next进程TSS描述符的选择符, (无用的)偏移地址`指令来进行任务切换。这种切换机制的特点正如实验手册所说：    
> 现在的Linux 0.11采用TSS和一条指令就能完成任务切换，虽然简单，但这条指令的执行时间却很长，在实现任务切换时需要200多个时钟周期(一个任务的时间片只有15个时钟周期)。而通过堆栈实现任务切换可能要更快，而且采用堆栈的切换还可以使用指令流水的并行优化技术，同时又使得CPU的设计变得简单。所以无论是Linux还是Windows，进程/线程的切换都没有使用Intel提供的这种TSS切换手段，而都是通过堆栈实现的。    
存在切换时间长，依赖CPU指令支持，单一指令切换无法使能指令流水的并行优化这些问题。而从课程的讲解我们知道：对于函数调用，依靠栈进行返回地址保存和弹栈返回操作；对于用户级线程，每个线程拥有一个线程控制块TCB，TCB关联着用户栈，TCB切换引起用户栈跟着切换，实现从一个线程切换到另一个线程以及再次切换回这个被换出的线程；对于核心级线程，线程切换发生在内核，从用户态进入内核态首先要发生线程用户栈到内核栈的切换，线程的TCB关联着内核栈，TCB切换引起内核栈切换，利用`iret`指令进行中断返回会引起内核栈中用户态参数的出栈，从而实现执行流程转移到新进程的用户态指令和用户栈。这些例子充分说明了栈在指令流程切换中的关键作用，再参照内核`main`函数完成初始化工作后，[以模拟特权级发生变化的内核中断返回的方式手动切换到任务0执行](#1.3 以模拟从特权级发生变化的内核中断处理过程返回的方式手动切换到任务0去执行)，完全可以自己想出是可以利用内核栈切换的方式实现进程切换的。    

#### 2. 是否还需要任务状态段TSS

