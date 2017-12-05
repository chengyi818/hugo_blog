---
layout: true
title: vim从入门到放弃_篇二
categories: 工具
tags: [tool, vim, emacs]
description: 编辑器是程序员最重要的工具,本系列讲述了我从vim入门到放弃vim的历程,希望对其他人有所帮助
photos:
- https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1491139173&di=1cc0bf82e240f8af1073ff2e4d43c8d6&imgtype=jpg&er=1&src=http%3A%2F%2Fwx3.sinaimg.cn%2Forj360%2F006B7ntWgy1fbumrmh4b9j31hc0u04qs.jpg
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
第一篇我们主要介绍了Vim的高效和优点.简而言之,就是Vim非常牛逼,值得学习.那么本篇我就我个人学习的经验来介绍下,如何配置和使用vim.

虽然我已经使用并学习vim两年时间了,但是vim还是会偶尔给我惊喜.能力有限,这里只是简单地介绍.

## 观点
### 能站在巨人的肩膀上,就不要站在平地
经常有新人寻求别人的配置,这个时候总会有一些人站出来说:
> vim的牛逼之处就在于它的高配置性,还是自己定制的最好等等

我的观点很明确既然已经有了那么好的配置,为什么还要自己折腾.

首先读懂别人的配置,然后汲取其中的优点,再去修改和定制.这样的效率远远高于自己在那里慢慢折腾.不要一上来就去修改,先使用一段时间.既然别人那样配置,又被人所广泛接受一定是有一定道理的.

额外的好处在于,如果大家都使用类似的配置,那么大家的快捷键也是类似的.这样就算别人帮你看代码也会方便一些.

## [spf13](https://github.com/spf13/spf13-vim)
spf13是我能想到的目前最优秀的配置.久经考验,值得拥有.

### 安装
安装非常简单,主要就是一行命令
```
    curl https://j.mp/spf13-vim3 -L > spf13-vim.sh && sh spf13-vim.sh
```
如果有其他安装问题,在Github也是很容易找到答案的.毕竟已经发布了多年,基本上你能遇到的问题,别人都已经遇到过了.

### 文档
在它的 [官网](http://vim.spf13.com/)有详细的介绍和文档.不要害怕英文,慢慢读一读.一边阅读一边使用,一定是可以理解的.

### 结构
本质上,vim的配置文件只有一个,就是`.vimrc`.spf13为了隔离变化,将之拆分成了三个文件,分别是:
#### 1. `.vimrc.before`
用于控制载入`.vimrc`前的一些设置
#### 2. `.vimrc.bundles`
用于控制配置的插件
#### 3. `.vimrc`
vim的主配置文件
以上的这三个文件是官方提供的,强烈不建议更改.

### 定制
那么,如果我们需要定制自己的配置怎么办呢?spf13早就帮你考虑好了.
如果仅仅是个人自己使用,只需要新建3个文件: `.vimrc.before.local`,`.vimrc.bundles.local`, `.vimrc.local`.
在这三个文件中,添加对应的内容,就可以修改了.
如果还希望再次发布给别人使用.将上面的`local`修改为`fork`即可.
关于这里的替换关系,只要稍微阅读下`.vimrc`和`.vimrc.before`就可以明白了.

### 插件
spf13本身配置了若干插件,使用了一种类似于插件包的配置手法.通过阅读`.vimrc.before`可以了解如何配置.
也可以参考我的 [配置](https://github.com/chengyi818/dotfiles/blob/master/home/.vimrc.before.fork)

### 参考
我的主要编码环境是c/c++/Python.如果有需要的也可以参考我的 [配置](https://github.com/chengyi818/dotfiles/tree/master/home)

## 尾记
使用vim的乐趣,在于不断尝试,不断探索.它总能给你一些新鲜的感受.无聊的时候,去读读官方的配置,总能发现新的快捷键.

使用新的快捷键总能发现新的乐趣,原来还可以支持这样的操作.多练习,多使用.把vim当成第一编辑器,总能学到更多的内容.晚安:moon:
