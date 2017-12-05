---
layout: true
title: vim从入门到放弃_篇四
tags: [tool, vim, emacs]
photos:
- https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1493727576&di=5f80d701abbfa52583d15eee30714c77&imgtype=jpg&er=1&src=http%3A%2F%2Fimg0.pconline.com.cn%2Fpconline%2F1211%2F23%2F3081921_3_thumb.jpg
date: 2017-04-23 22:07:58
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
前面我们介绍了*Spacemacs*的种种优点,本篇我们来介绍下*Spacemacs*的入门.我自己接触*Spacemacs*大概有半年时间了,那么我自己精通了吗?恐怕还没有.因此本文只是抛砖引玉,更多精彩的内容等待你去挖掘.所有的工具都是这样的,一定要多用,这样就会慢慢熟悉.

## 安利
如果我前面的文章还是不能引起你的兴趣,欢迎你阅读陈斌的这篇[一年成为Emacs高手](http://blog.csdn.net/redguardtoo/article/details/7222501/).

## 安装
首先第一步,一定是安装软件.这部分我不再赘述,请参考[github spacemacs主页](https://github.com/syl20bnr/spacemacs)

## 官方文档
任何软件工具,官方文档都是仅次于源码的权威资料,*Spacemacs*也不例外.况且正如我之前所说,*Spacemacs*非常重视用户的友好性.因此它有大量的文档是关于引导你入门的.

阅读文档的方式有两种:
1. 你在项目官网找到文档的[链接](http://spacemacs.org/),通过浏览器阅读.
2. 另外,如果你已经成功安装了*Spacemacs*,也可以通过快捷键`SPC h SPC`来打开文档,其中`SPC`表示空格键.事实上,*Spacemacs*的大部分快捷键都可以`SPC`来打开.

文档是用英文书写的,在有能力的情况下,还是尽量阅读官方文档.因为*Spacemacs*还在高速更新当中,只有官方文档能够保持及时更新,而其他的介绍文章包括本文往往不够及时.

## 常用快捷键
下面我来介绍一些最为常用的快捷键.
### 约定
1. `SPC`表示空格
2. `C`表示ctrl键

### 帮助
首先我们来看看与帮助功能相关的快捷键.*Spacemacs*内置的帮助功能强大而易用.
1. `SPC h SPC`查看所有文档.可以通过方向键或者`C-j/C-k`来选择文档.
2. `SPC h d b`查询快捷键,如果你想实现某个功能却不知道快捷键的话,可以使用它.比如分割屏幕,只知道`split`和`window`来个单词,就可以搜索出所有相关快捷键了.
3. `SPC h d f`查询函数.
4. `SPC h d k`查询快捷键对应的函数.比如我不知道`SPC w v`的作用,在输入`SPC h d k`之后接着输入`SPC w v`就可以看到相应的解释了.
5. `SPC ?`功能与第二项类似.

有了这些帮助功能,在有需要的帮助的时候,就可以及时查询了.
### 配置文件管理
1. `SPC f e d`打开配置文件
2. `SPC f e R`同步配置文件

### 窗口管理
1. `SPC w p m`查看log
2. `SPC  0..9`切换窗口
3. `SPC f t`打开当前文件所在的目录
4. `SPC w s`水平分割窗口
5. `SPC w v`垂直分割窗口
6. `SPC w c`关闭当前窗口

### 文件管理
1. `SPC f f`打开文件（夹）
2. `SPC  /`智能搜索
3. `SPC f R`重命名当前文件
4. `SPC b d`关闭当前 buffer
5. `SPC f r`最近打开文件
6. `SPC b b`最近打开的buffer

### 项目管理
1. `SPC p t`打开当前文件所在的根目录
2. `SPC p p`切换项目
3. `SPC p D`在 dired 中打开项目根目录
4. `SPC p f`在项目中搜索文件名
5. `SPC p R`在项目中替换字符串

更多的快捷键,在`SPC`键稍加暂停,即可看到菜单提示,多用多尝试.
## Trouble Shooting
在迁移到*Spacemacs*的过程后,因为卡死的问题导致我不能正常工作,不得不迁回*vim*.在询问了[zilongshanren](https://github.com/zilongshanren)后,目前我知道的解决卡死的方法如下:
1. `C-g`.大部分卡死问题都可以通过`Ctrl-g`来解决.
2. 向Emacs进程发送`SIGUSR2`.在Linux上命令为`kill -SIGUSR2 <pid of emacs>`.

## 进阶玩法
### [Spacemacs layers](https://github.com/syl20bnr/spacemacs/tree/master/layers)
*Spacemacs*提供了大量的layer,可以根据需要来进行选择.其中比较有用的包括:
1. [git](https://github.com/syl20bnr/spacemacs/tree/master/layers/%2Bsource-control/git)git插件
2. [org](https://github.com/syl20bnr/spacemacs/blob/master/layers/%2Bemacs/org/README.org)Emacs的特色功能
3. [chinese](https://github.com/syl20bnr/spacemacs/blob/master/layers/%2Bintl/chinese/README.org)中文输入法
4. [cscope](https://github.com/syl20bnr/spacemacs/blob/master/layers/%2Btags/cscope/README.org)代码跳转
当然更多有意思的layer等着你自己去挖掘.

### Elisp
使用*Emacs*完全不需要掌握*Emacs lisp*,但是要玩好*Emacs*一定是要掌握*Emacs lisp*的.

### 个性化定制
与*Vim*类似,在掌握了基本用法后,你会根据自己的需要自由的定制配置.

## 参考资料
1. *zilongshanren*为*Spcemacs*的推广做了很多工作.[Spacemacs-rocks](https://github.com/emacs-china/Spacemacs-rocks)包含了两季的视频.

2. [Emacs Wiki](https://www.emacswiki.org/emacs/EmacsWiki)是最权威的Emacs资料来源地.

3. [Emacs China](https://emacs-china.org/)同样可以找到很多帮助,里面的坛友非常热心和Geek.

## 总结
对这个世界充满好奇心,对提高效率有执着的追求.

正如我之前所说的只要你充满着Geek精神,总有一天你会遇到*Vim*,你会遇到*Emacs*.因为他们都是那么的优秀,历经风雨依然散发着迷人的魅力.

Happy Hacking:grinning:
