---
layout: post
title: openwrt编译过程概述
categories: openwrt
tags: [openwrt,makefile,深度]
comments: true
analytics: true
---

## 前言
又到了成胖子每周一博的时间了,本周是第三周.
本周我们继续了解`openwrt`的编译过程,如果还有没写过简单ipk或者编译过openwrt的朋友,可以参见我之前的[博客](http://blog.csdn.net/icy_river/article/details/48260859)或者网上的其他[文章](https://www.baidu.com/s?wd=%E7%BC%96%E8%AF%91openwrt&rsv_spt=1&rsv_iqid=0x8f1fcaec0016b4fc&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_sug3=15&rsv_sug1=10)

<!--more-->

## 一 年轻的冲动
为什么我在学习的过程中,有先学习整个编译过程和Makefile的冲动呢?
> 1.我们知道电脑的运算速度是很快的.即使如此,一个完整的编译过程往往需要好几个小时.这中间到底发生了些什么?屏幕上一闪而过的像天书一样的东西,我怎么才能有所了解?
>
2.经历漫长的等待,我们得到了一个镜像bin文件.这里面到底包含了什么东西?我可以知道么?
>
3.编译单个ipk的时候,我按照模板写的Makefile怎么和我在书上见的长的不太一样呢?
>
4.在单个ipk中,怎么包含头文件,动态库,静态库?怎么解决依赖关系?
>
5.在输入`make menuconfig`之后,又发生了什么?弹出的图形界面中,怎么会有我放在`package`目录中的源码的信息?
>
6.有的时候,编译报错了.提示信息它认识我,我不认识它.我要怎么排查呢?
>
...

零零散散的总是有很多疑问困扰着我.我相信有很多刚接触`openwrt`的朋友都和我有同感.而所有这些都是可以通过完整学习编译过程来解答的.这样想想是不是更有学习的动力了呢?

## 二 学习曲线
### 2.1 Makefile基本语法
&emsp;我们知道编译过程是通过Makefile来控制的.这样而言,Makefile的基本语法就必须有所了解.网上有很多零散的资料往往不成体系.推荐阅读一个是陈皓自己写的和翻译的GNU Make的[手册](http://blog.csdn.net/haoel/article/details/2886).还是比较全的.英文还凑合的朋友,我推荐还是尽力读读官方的[手册](https://www.gnu.org/software/make/manual/),简介精炼,没事学点英文也是好的.后面的文章,假定读者对Makefile语法有所了解.

### 2.2 bash基础
Makefile中的执行部分(recipe)是有bash脚本组成的,所以我们同时应当对bash shell有所了解.

### 2.3 world
有了这两部分的预备知识,我们将开始我们的征途.我们的目标是**world**.

## 三 编译过程概述

###3.1 主机预装工具

在编译源码之前,我们必须手动安装一部分工具 .

```
sudo apt-get install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip sharutils subversion libncurses5-dev ncurses-term zlib1g-dev
```

这部分是在执行编译工作之前的.

### 3.2 编译host工具
除了我们在第一步安装的工具,编译过程中还需要其他一些主机工具.这部分工具将首先编译.

### 3.3 编译交叉工具链
`openwrt`自带交叉编译链,当然在编译目标平台软件前,需要先编译.

### 3.4 编译内核模块
因为部分内核模块将会生成独立的ipk,所以内核模块需要首先编译.

### 3.5 编译ipk
这里将编译`package`目录下的各个软件包,这也是和我们最为息息相关的.之后的博客将会重点介绍这个部分.

### 3.6 安装ipk
将生成的ipk安装到文件系统之中(比如build_dir/target-XXX/root-ramips目录).

### 3.7 编译内核
在完成ipk编译之后,将会编译内核,压缩内核.同时使用mkimage工具,在内核前面生成一个用于uboot识别的头部.

### 3.8 合成
在最后一步,将squashfs格式的文件系统和内核连接在一起,即生成了目标镜像文件.

## 尾声
本周到此为止,下一篇我们将介绍根目录下的Makefile,从而知道为什么编译的大致过程是上面提及的八步.
