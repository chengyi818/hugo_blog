---
title: "Whole_archive"
date: 2022-03-10T21:46:23+08:00
lastmod: 2022-03-10T21:46:23+08:00
draft: true
description: "whole-archive的作用"
tags: ["Linker"]
categories: ["Compiler"]

---

# 前言
链接静态库时,默认情况下会将引用到的符号所在的.o全部链接进来.

这句话有两个含义:
1. 如果一个.o中的所有符号均未被引用到,那么默认情况下,这个.o不会被链接.
2. 如果一个.o中有一个符号被引用到,哪怕其他符号未被引用,也会被链接进来.

在某些场景下,我们希望整个.o,无论是否被引用到都链接进来.这个时候可以使用`--whole-archive`.具体方法可以参考ref.



<!--more-->












----

# ref
[gcc链接参数--whole-archive的作用](https://cloud.tencent.com/developer/article/1403300)
