---
layout: true
title: vim从入门到放弃_篇三
tags: [tool, vim, emacs]
photos:
  - https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1491919879&di=783431702b21b25ae2b3839bc7c0d0f5&imgtype=jpg&er=1&src=http%3A%2F%2Fpic.lvmama.com%2Fuploads%2Fpc%2Fplace2%2F2016-05-29%2F60504337-83de-47dc-a422-04966772cf50.jpg
date: 2017-04-04 20:34:58
categories: 工具
description: 编辑器是程序员最重要的工具,本系列讲述了我从vim入门到放弃vim的历程,希望对其他人有所帮助
---

阅读条件:
> 编辑器爱好者,了解vim基本使用

---

篇一: [初识vim](http://yitinglove.cn/blog/2017/03/19/vim_to_emacs_1/)
篇二: [vim的配置与使用](http://yitinglove.cn/blog/2017/03/26/vim_to_emacs_2/)
篇三: [vim的不足和新的选择](http://yitinglove.cn/blog/2017/04/04/vim_to_emacs_3/)
篇四: [spacemacs入门](http://yitinglove.cn/blog/2017/04/23/vim_to_emacs_4/)

<!--more-->

## 前言
前面两篇主要介绍了vim的一些优势.本篇主要是推荐大家去试试Emacs的一个发行版[Spacemacs](http://spacemacs.org/).

关于Vim和Emacs孰优孰劣的争论已经非常多了,但是很少有人能够同时精通两种编辑器,大部分人都是在捍卫自己的选择.

国内Emacs的用户要远远少于Vim,所以推荐Vim用户可以去了解了解Emacs.虽然有一些学习成本,但是我觉得物有所值.

## Vim的精髓
vim的精髓在于模式切换,简而言之,在插入模式下,按键的作用是输入.但是在普通模式下,按键又被赋予了大量特殊功能.

## Vim的不足
### 遗忘的快捷键
我个人感觉我已经算是比较重度的vim用户了,却依然有大量记不住的快捷键.然而人脑的记忆模型就是这样的,长时间不使用的东西,自然就会发生遗忘.

那么当我想使用某个快捷键时,能不能快速的搜索呢?或者有什么助记的方法呢?

### 扩展语言
`vimscript`是一种小众的脚本语言,虽然我没有详细研究过.但是看了各种介绍后,我感觉对我的吸引力不大.

### 重叠的插件
在vim中,经常发生一件工作好几个插件重叠的情况.比如Python语法检查,`YouCompleteMe`,`Syntantic`,`Python-mode`都会涉及.有时真的搞不太清楚,到底是谁在其作用.

## Emacs的优势
### 强大的扩展语言
Emacs的扩展语言是久闻大名的Emacs lisp -- 著名的函数式编程语言的鼻祖.我对lisp非常感兴趣.从我现在的观点来看,Vim和Emacs完全不冲突.Emacs完全可以通过`Evil`插件来借鉴Vim的模式切换.

正是由于Emacs强大的扩展和定制性,所以Emacs才会被称为神之编辑器 -- 无所不能.

## Spacemacs
对于从vim迁移到Emacs的同学,我个人强烈推荐,我自己正在使用的[spacemacs](http://spacemacs.org/).目前已经在[Github](https://github.com/syl20bnr/spacemacs)上有了11000+的star了.

### 切换成本低
通过默认的`Evil`插件,基本实现了绝大多数Vim的功能,稍微熟悉就可以上手工作.我们将在下篇介绍Spacemacs的快速入门.

### 快捷键解释和助记
通过空格键可以呼出快捷键菜单,通过有意义的分类后,你会发现快捷键非常容易查找.比如`空格+w`可以和窗口(window)相关的快捷键.

另过,也可以通过`空格+h`中的各种帮助来查找快捷键,甚至是解释某个快捷键的功能.

### 插件包的引入
layer是指一个功能相关的package的集合.通过社区维护的layer的引入,我们可以通过快速引入一个layer来实现插件功能,而不需要去分别配置.

其实如果阅读过spf13配置的同学,已经可以看到spf13已经使用了类似的概念.但是这是配置上的概念,没有社区为你维护.相对来讲,spacemacs的layer要更为成熟.

### 其他更强大的功能
Emacs还可以实现一些更为强大的功能,有些对我比较有吸引力.比如使用org-mode实现GTD.比如番茄钟功能的引入.比如可以直接使用有道词典查单词.

## 尾记
对于Spacemacs我也是刚刚入门,希望大家不要抱着偏见来看这些编辑器.Vim的模式切换已经深入人心,对于Vim的学习成本不会成为沉没成本.

怀着一颗好奇心去试试Spacemacs,说不定有新的惊喜呢 :)
