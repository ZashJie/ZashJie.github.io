---
title: xv6 源码解析 - allocproc 函数
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
    // Look in the process table for an UNUSED proc.
    // If found, initialize state required to run in the kernel,
    // and return with p->lock held.
    // If there are no free procs, or a memory allocation fails, return 0.
    static struct proc*
    allocproc(void)
    {
    struct proc *p;

    for(p = proc; p < &proc[NPROC]; p++) {
        acquire(&p->lock);
        if(p->state == UNUSED) {
        goto found;
        } else {
        release(&p->lock);
        }
    }
    return 0;

    found:
    p->pid = allocpid();

    // Allocate a trapframe page.
    if((p->trapframe = (struct trapframe *)kalloc()) == 0){
        release(&p->lock);
        return 0;
    }

    // An empty user page table.
    p->pagetable = proc_pagetable(p);
    if(p->pagetable == 0){
        freeproc(p);
        release(&p->lock);
        return 0;
    }

    // Set up new context to start executing at forkret,
    // which returns to user space.
    memset(&p->context, 0, sizeof(p->context));
    p->context.ra = (uint64)forkret;
    p->context.sp = p->kstack + PGSIZE;

    return p;
    }
    ```

### 解析

##### 函数逻辑
从 64 个进程中找到一个没有被使用的函数，找到后 分配 进程号 `pid`，给这个进程的 `trapframe` 分配存储空间，再给这个进程分配进程页表，

##### `trapframe` 结构
这个结构用于在内核状态切换时，将用户态进程的当前状态进行保存到 `trapframe` 中，在内核处理完后，将用户态进程的状态进行恢复，所以专门为 这个状态分配了一个存储空间

##### 调用的函数
[[xv6 源码解析 - proc_pagetable 函数]]

