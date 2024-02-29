---
title: xv6 源码解析 - vmcreate 函数
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
  // create an empty user page table.
  // returns 0 if out of memory.
  pagetable_t
  uvmcreate()
  {
    pagetable_t pagetable;
    pagetable = (pagetable_t) kalloc();
    if(pagetable == 0)
      return 0;
    memset(pagetable, 0, PGSIZE);
    return pagetable;
  }
  ```

### 解析

##### 函数流程
通过调用 `kalloc` 函数获取到没有被使用到的物理内存，并强制类型转换成 pagetable_t，并作为返回值返回

##### 需要多思考的点
函数分配的内容大小为 `4096 bytes` 但是能够充当 页表目录，也能够当做某进程的物理内存来使用

##### 涉及到的代码
[[xv6 源码解析 - kalloc 函数]]
[[xv6 源码解析 - pagetable_t]]
