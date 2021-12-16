---
title: "Pid_tid_pgid_tgid_sid"
date: 2021-12-13T22:52:11+08:00
lastmod: 2021-12-13T22:52:11+08:00
draft: true
description: "梳理Linux中进程相关各种ID的概念"
tags: ["PID"]
categories: ["kernel"]

---

# 引言
Linux中有进程相关的ID概念包括: 线程ID(tid), 进程id(pid), 父进程(ppid), 进程组id(pgid), 会话id(sid)等等概念.
参考资料中的两篇博客都写的很好,也很详细.本来也没有什么需要补充的,这里我从内核系统调用的角度看看这些信息都承载在哪里,表征了哪些信息

<!--more-->

# task_struct
从操作系统的基本概念上看,进程是资源分配的最小单元,线程是调度的最小单元,即各线程间共享进程资源. 对于部分操作系统而言,会区分进程控制结构体(PCB, Process control block)和线程控制结构体(TCB, thread control block). 对于Linux而言没有做这样的区分,直接使用了`task_struct`表示线程结构体;属于同一个进程的线程,在`task_struct`中指向相同的资源.

在`task_struct`中,我们能明显看到几个和进程id相关的字段.

```
// kernel5.10.84/include/linux/sched.h
struct task_struct {
    pid_t pid;
    pid_t tgid;
    struct task_struct *real_parent;
    struct task_struct *parent;
	struct task_struct		*group_leader;
    ...
}
```

这个结构体,我们稍微再过来摸索. 下面我们顺着系统调用的调用流程来看看这些id.

# 系统调用
进程各种id的获取与设置函数的实现都在`kernel/sys.c`中.下面我们从简单地一个个来看.

## getpid

## getppid

## getpgid

## getsid

## gettid

## getpgrp


# 参考资料
1. [The Linux Process Principle，NameSpace, PID、TID、PGID、PPID、SID、TID、TTY](https://www.cnblogs.com/LittleHann/p/4026781.html)
2. [linux内核之进程的基本概念(进程，进程组，会话关系）](https://www.cnblogs.com/zengyiwen/p/5755191.html)
3. [Linux task_struct parent 和 real_parent 的区别](https://blog.csdn.net/jektonluo/article/details/49736297)
