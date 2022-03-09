---
title: "Arm64_asid"
date: 2022-03-09T20:28:28+08:00
lastmod: 2022-03-09T20:28:28+08:00
draft: false
description: "原来这就是arm的asid"
tags: ["Arm64", "MMU", "Cache"]
categories: ["Os"]

---

最近在工作中定位了一个比较困难的问题,涉及到了Arm64的asid相关的概念,这里来简单总结下它的来龙去脉.


<!--more-->

# ASID在解决什么问题
作为现代的CPU基本都有MMU(Memory Management Unit)用于虚拟地址和物理地址的转换,这里我们不再赘述. 而这个转换的过程由于需要遍历多级页表,其速度是比较慢的. 那么这里为了加速转换的过程,硬件提供了一个缓存即,TLB. 当需要虚拟地址(VA)向物理地址(PA)转换时,则先去TLB(translation lookaside buffer)查询,如果命中了则直接返回物理地址,否则去遍历页表,同时将VA和PA的对应关系保存到TLB中.

但是我们知道用户态的程序拥有各自独立的虚拟地址空间,其对应的物理地址并不相同. 如果仅仅使用虚拟地址,那么相同的VA必然对应相同的PA. 因此ARM64,就设计了ASID(Address Space ID).

如下图所示,比较好的说明了这个问题. TLB中缓存是区分全局或者局部的, 对于内核空间地址,所有进程都一样.因此不需要ASID参与映射. 对于用户空间地址,则需要ASID参与.
![ASID](https://cdn.jsdelivr.net/gh/chengyi818/images@master//img/f77c9be0d1c9476189e07edfbd098cc1.png)


# 其他细节
1. 如通过`TCR_EL1.AS`寄存器配置ASID的宽度, `TCR_EL.A1`配置ASID是从TTBR0或者TTRB1读取.
2. 在linux kernel中,如果ASID配置了宽度为8bit,则总可用的ASID只有256个.进程那么多是怎么分配的呢?

---

# Ref
1. [ARM64中的ASID地址空间标识符](https://blog.csdn.net/weixin_42314225/article/details/120210638)
2. [ASID](https://zhuanlan.zhihu.com/p/55265099)
3. [Linux kernel ASID](https://blog.csdn.net/tiantao2012/article/details/82756686)
4. [进程切换分析_2: TLB处理](http://www.wowotech.net/process_management/context-switch-tlb.html)
