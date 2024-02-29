---
title: xv6 源码解析 - procinit 函数
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
  // kernel/proc.c
  // initialize the proc table at boot time.
  void
  procinit(void)
  {
    struct proc *p;
    
    initlock(&pid_lock, "nextpid");
    for(p = proc; p < &proc[NPROC]; p++) {
        initlock(&p->lock, "proc");

        // Allocate a page for the process's kernel stack.
        // Map it high in memory, followed by an invalid
        // guard page.
        char *pa = kalloc();
        if(pa == 0)
          panic("kalloc");
        uint64 va = KSTACK((int) (p - proc));
        kvmmap(va, (uint64)pa, PGSIZE, PTE_R | PTE_W);
        p->kstack = va;
    }
    kvminithart();
  }
  ```
### 解析

##### 函数调用地点
在 `main.c` 中被调用，在内核初始化完，调用 `procinit` 函数进行映射
##### 逻辑
为每个进程分配一个 4096 的物理地址，作为内核栈
##### 为什么分配内核栈需要不同的虚拟地址呢
因为为了防止相同的虚拟地址映射，也为了保证隔离性和安全性；
内核的虚拟地址映射都是映射在 同一个内核空间中。
