---
title: webserver中 利用链表存放定时器轮询关闭空闲线程
date: 2023-03-11 10:34:00 +0800
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

## 定时器容器
是一个带头尾结点的升序双向链表，并按照超时时间进行排序（超时时间短到长）

###### 构造函数
为头尾节点赋nullptr

###### 析构函数
将链表中每个节点delete掉

###### 添加定时器
- 如果定时器为空，直接返回
- 如果头节点为空，那么将定时器存入头尾节点
- 上两条件都不满足，那么将定时器按顺序插入到链表中
- 如果上三都不满足，那么定时器的超市时间比较长，处于尾节点，进行特殊处理

###### 调整定时器
当某个定时器发生变化的时候，调整定时器在链表中的位置
- 如果被调整的定时器在链表尾部或者仍然小于下一个定时器的超时值则直接返回
- 若为头结点或内部结点，则将定时器取出重新插入

###### 删除定时器
- 同时为头尾结点（当链表中只有一个定时器时）
- 为头结点时
- 为尾结点时
- 剩下则是内部结点


#### 定时处理函数
计时进行轮询，因为链表里面的定时器是升序排列的，所以从上到下去遍历定时器，将到期的定时器调用[[回调函数]]，执行定时事件，并删除

##### 定时器的超时时间

`timer->expire = curTime + 3 * TIMESLOT  (TIMESLOT = 5)` 所以超时时间是15s    

##### `adjust_timer` 源码解析

当收到读写操作时，更新定时器，将定时器的超时时间进行调整
	```C++
	void Webserver::adjust_timer(util_timer *timer)
	{
		time_t cur = time(NULL);
		timer->expire = cur + 3 * TIMESLOT;
		utils.m_timer_lst.adjust_timer(timer);

		LOG_INFO("%s", "adjust timer once");
	}
	```
将定时器在容器中的位置同样进行调整
	```C++
	void sort_timer_lst::adjust_timer(util_timer *timer) {
		if (!timer) {
			return;
		}
		util_timer *tmp = timer->next;
		if (!tmp || (timer->expire < tmp->expire)) {
			return;
		}
		// 当被调整的是头结点，将头结点取出，放入add_timer中
		if (timer == head) {
			head = head->next;
			head->prev = nullptr;
			timer->next = nullptr;
			add_timer(timer, head);
		}
		else {
			timer->prev->next = timer->next;
			timer->next->prev = timer->prev;
			add_timer(timer, timer->next);
		}
	}
	```
