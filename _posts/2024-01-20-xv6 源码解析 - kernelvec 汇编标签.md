---
title: xv6 源码解析 - kernelvec 汇编标签
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
  ```c
    #
          # interrupts and exceptions while in supervisor
          # mode come here.
          #
          # push all registers, call kerneltrap(), restore, return.
          #
  .globl kerneltrap
  .globl kernelvec
  .align 4
  kernelvec:
          // make room to save registers.
          addi sp, sp, -256

          // save the registers.
          sd ra, 0(sp)
          sd sp, 8(sp)
          sd gp, 16(sp)
          sd tp, 24(sp)
          sd t0, 32(sp)
          sd t1, 40(sp)
          sd t2, 48(sp)
          sd s0, 56(sp)
          sd s1, 64(sp)
          sd a0, 72(sp)
          sd a1, 80(sp)
          sd a2, 88(sp)
          sd a3, 96(sp)
          sd a4, 104(sp)
          sd a5, 112(sp)
          sd a6, 120(sp)
          sd a7, 128(sp)
          sd s2, 136(sp)
          sd s3, 144(sp)
          sd s4, 152(sp)
          sd s5, 160(sp)
          sd s6, 168(sp)
          sd s7, 176(sp)
          sd s8, 184(sp)
          sd s9, 192(sp)
          sd s10, 200(sp)
          sd s11, 208(sp)
          sd t3, 216(sp)
          sd t4, 224(sp)
          sd t5, 232(sp)
          sd t6, 240(sp)

    // call the C trap handler in trap.c
          call kerneltrap

          // restore registers.
          ld ra, 0(sp)
          ld sp, 8(sp)
          ld gp, 16(sp)
          // not this, in case we moved CPUs: ld tp, 24(sp)
          ld t0, 32(sp)
          ld t1, 40(sp)
          ld t2, 48(sp)
          ld s0, 56(sp)
          ld s1, 64(sp)
          ld a0, 72(sp)
          ld a1, 80(sp)
          ld a2, 88(sp)
          ld a3, 96(sp)
          ld a4, 104(sp)
          ld a5, 112(sp)
          ld a6, 120(sp)
          ld a7, 128(sp)
          ld s2, 136(sp)
          ld s3, 144(sp)
          ld s4, 152(sp)
          ld s5, 160(sp)
          ld s6, 168(sp)
          ld s7, 176(sp)
          ld s8, 184(sp)
          ld s9, 192(sp)
          ld s10, 200(sp)
          ld s11, 208(sp)
          ld t3, 216(sp)
          ld t4, 224(sp)
          ld t5, 232(sp)
          ld t6, 240(sp)

          addi sp, sp, 256

          // return to whatever we were doing in the kernel.
          sret
  ```
### 解析
##### 栈指针下移
`addi sp, sp, -256` 将当前的栈指针往下移动256，腾出256的空间来保存寄存器
##### 存储
通过 `sd` ，汇编指令将寄存器的值分别存储在栈的对应空间
##### 调用 kerneltrap

##### 加载
通过 `ld` 汇编指令将之前存储的数据加载回寄存器中

##### 结束返回处理
`addi sp, sp, 256` 将栈指针回到原来的指向

最后通过 `sret` 返回