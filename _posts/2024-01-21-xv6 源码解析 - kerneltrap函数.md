---
title: xv6 源码解析 - kerneltrap函数
date: 2024-01-21 10:34:00 +0800
categories: [计算机]
tags: [操作系统/6S081/源码解析]
pin: false
author: ZashJie

toc: true
comments: true
typora-root-url: ../../ZashJie.github.io
math: false
mermaid: true


---

#操作系统/6S081/源码解析 
### code
```c
// interrupts and exceptions from kernel code go here via kernelvec,
// on whatever the current kernel stack is.
void 
kerneltrap()
{
  int which_dev = 0;
  uint64 sepc = r_sepc();
  uint64 sstatus = r_sstatus();
  uint64 scause = r_scause();
  
  if((sstatus & SSTATUS_SPP) == 0)
    panic("kerneltrap: not from supervisor mode");
  if(intr_get() != 0)
    panic("kerneltrap: interrupts enabled");

  if((which_dev = devintr()) == 0){
    printf("scause %p\n", scause);
    printf("sepc=%p stval=%p\n", r_sepc(), r_stval());
    panic("kerneltrap");
  }

  // give up the CPU if this is a timer interrupt.
  if(which_dev == 2 && myproc() != 0 && myproc()->state == RUNNING)
    yield();

  // the yield() may have caused some traps to occur,
  // so restore trap registers for use by kernelvec.S's sepc instruction.
  w_sepc(sepc);
  w_sstatus(sstatus);
}
```
### 解析

读取并保存寄存器对应数据
`if((sstatus & SSTATUS_SPP) == 0)` 如果原先的状态为0，也就是用户态就有问题；要保证先前为监管者模式
`if(intr_get() != 0)` 检查中断是否启用，需要保证系统 不能启用中断
在保证中断被禁用后，我们要检查是否还有隐型的中断发生，通过 `if((which_dev = devintr()) == 0)`  判断（2为时钟中断，1为其它中断，0为未识别的中断），如果是未识别的中断，那么报错
`if(which_dev == 2 && myproc() != 0 && myproc()->state == RUNNING)` 如果是时钟中断，并且当前进程可用，那么 调用 `yield()` 函数，放弃CPU进行一轮调度
那么在调度的时候，可能会有陷阱的发生，存储pc跟status状态的寄存器可能被覆盖了，所以要重新将一开始存储的数据恢复

##### 总结
我认为这个函数对于系统调用是保证系统稳定性的一个过程