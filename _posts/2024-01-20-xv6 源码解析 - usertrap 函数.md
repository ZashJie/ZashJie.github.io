---
title: xv6 源码解析 - usertrap 函数
date: 2024-01-20 10:34:00 +0800
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
  ```C
  //
  // handle an interrupt, exception, or system call from user space.
  // called from trampoline.S
  //
  void
  usertrap(void)
  {
    int which_dev = 0;

    if((r_sstatus() & SSTATUS_SPP) != 0)
      panic("usertrap: not from user mode");

    // send interrupts and exceptions to kerneltrap(),
    // since we're now in the kernel.
    w_stvec((uint64)kernelvec);

    struct proc *p = myproc();
    
    // save user program counter.
    p->trapframe->epc = r_sepc();
    
    if(r_scause() == 8){
      // system call

      if(p->killed)
        exit(-1);

      // sepc points to the ecall instruction,
      // but we want to return to the next instruction.
      p->trapframe->epc += 4;

      // an interrupt will change sstatus &c registers,
      // so don't enable until done with those registers.
      intr_on();

      syscall();
    } else if((which_dev = devintr()) != 0){
      // ok
    } else {
      printf("usertrap(): unexpected scause %p pid=%d\n", r_scause(), p->pid);
      printf("            sepc=%p stval=%p\n", r_sepc(), r_stval());
      p->killed = 1;
    }

    if(p->killed)
      exit(-1);

    // give up the CPU if this is a timer interrupt.
    if(which_dev == 2)
      yield();

    usertrapret();
  }

  ```

### 解析
首先获取当前的寄存器状态，观察SPP位是否为0（观察之前的状态是否为用户态），确认之前为用户态后继续
##### 更改 `STVEC` 寄存器
`w_stvec((uint64)kernelvec);` 将`kernelvec`汇编标签写入到 `stvec` 寄存器中，该寄存器是专门用来存储中断和异常的向量表（向量表指的是一系列处理程序的地址）

##### 保存用户模式 pc 值
然后获取当前进程，用 `p->trapframe->epc` 保存当前用户模式下的程序地址（保存的原因是 `usertrap` 中可能有进程切换，可能导致`sepc`被覆盖）

##### 判断
`if(r_scause() == 8)` 如果当前的触发的原因是因为系统调用的话，
	先判断当前进程是否被杀死了，被杀死了就退出；将 `p->trapframe->epc` 的程序地址往前一个字节，让其返回时不是在进入的地址，而是进行地址的下一条指令；
	然后开启中断，这里开启中断的原因是为了保证在进入系统调用前，寄存器状态不会受到其它中断状态影响，在进入调用前开启是为了，在漫长的系统调用中，能够处理别的系统调用的发生
`else if((which_dev = devintr()) != 0)` 如果识别的原因的话，跳过
`else` 如果二者都不属于，是一个异常，故终止进程

`usertrapret();` 通过调用该函数返回到用户模式下

##### 设计的代码
[[xv6 源码解析 - risc.h#`sstatus_spp`]]