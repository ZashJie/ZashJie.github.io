---
title: 内存泄漏与C++
date: 2023-03-10 10:34:00 +0800
categories: [计算机]
tags: [语言/CPP]
pin: false
author: ZashJie

toc: true
comments: true
typora-root-url: ../../ZashJie.github.io
math: false
mermaid: true


---

#项目/WebServer/单Reactor #语言/CPP 
### SIGTERM 信号
当进程接收到 SIGTERM 信号时，将 标志是否关闭服务的标志位 置为true，结束循环

### addsig() 函数

在webserver中加上特定的信号，不执行它们的默认操作（[[信号#执行默认操作跟执行信号的区别]]），而是要在项目中捕获到这个特定信号后，去处理相应的操作，在webserver中特别捕获了两个信号分别是 `SIGALRM` 跟 `SIGSTERM` 

### 定时发送 SIGALRM 信号

	```C++
	//定时处理任务，重新定时以不断触发SIGALRM信号
	void Utils::timer_handler()
	{
		m_timer_lst.tick();
		alarm(m_TIMESLOT);
	}
	```

##### 目的是
周期性的发送SIGALRM信号

### 怎么处理
在我的webserver项目中主要通过 `addsig` 函数关注两个信号 分别是 `SIGALRM` 跟 `SIGTERM` 信号，对这两个信号进行对应的处理，不让它们让我的项目执行 信号的默认操作，而是执行自定义的行为，分别是关闭定时器重新定时以及去将升序链表中的所有定时器进行一个处理 跟 关闭webserver操作