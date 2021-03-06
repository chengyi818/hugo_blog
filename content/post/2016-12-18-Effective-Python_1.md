---
title: "把Effective Python读薄之一"
tags: ["python"]
date: 2016-12-18T19:16:56+08:00
---
这两天看到一篇博客,名为<把编程珠玑读薄>,是为名.

本文是我总结的Effective Python的第一章

阅读条件:
> 掌握Python基本语法

<!--more-->

## 用Pythonic的方式思考

Python程序员崇尚美观易读的代码,准则我想大概有两个:

1. `import this`
> Python之禅中,每一句都应当时时牢记,并向之接近

2. 我们都是成年人
> Python要求程序员为自己的行为负责,并不赞同在编程时过多地限制行为


### 1.Python版本

在16年年末看来,Python3毫无疑问是未来的方向和趋势.Python2和Python3的不兼容给Python程序员带来的巨大的麻烦,但程序员不就是要拥抱变化么.

所以尽快地切换到Python3是明智之选

### 2.代码风格

代码风格的重要性毋庸置疑,Python主要遵守PEP8.

我目前在编辑器中使用Pylint作为检查工具,只要将Pylint提示的问题都做到心中有数,代码风格应该问题不大.

### 3.字符串编码

Python的编码问题也是老生常谈了,相关文章也很多了,不再赘述.

在Linux环境下,将默认编码统一为**UTF-8**,是一个良好的习惯.

### 4.辅助函数

Python语法支持很多奇异的特性,比如列表生成式等等.过多地滥用这些特性,会把语句写的非常晦涩难懂.

前面我们就提到的Python的原则,代码的易读和美观在Python中是非常重要的.

所以书中推荐使用辅助函数来替代一些晦涩难懂的语句.

### 5.切片操作

序列,尤其是列表(list)的切割在Python中非常常用,使用方法灵活且安全.

推荐使用

### 6.切片操作注意

在单次切片操作中,不要同时指定start,end,stride.

这会影响易读性,如果确实需要使用,不如分成两步

### 7.列表生成式

列表生成式(list comprehension)是Python中非常常见的用法

推荐使用

### 8.列表生成式注意

和切片操作的注意类似,虽然列表生成式支持嵌套,但是不要忘了在Python中代码的易读和美观非常重要.

通常如果列表生成式嵌套不要超过两层

### 9.列表生成式无力处理大数据量

列表生成式会将数据一次性全部载入内存中,对于大数据量而言,这是系统无法承受的.

所以在遇到大数据量,推荐使用生成器.

### 10.使用enumerate取代range

使用索引访问序列,是C/C++程序员的常规做法.

在Python中,我们更加推荐使用enumerate,更加直观.

### 11.zip同时遍历两个迭代器

Python内置的zip函数可以同时遍历多个迭代器,相较于使用下标遍历更为直观.

换言之,Python为程序员提供了很多内置的函数,通常更加高效和直观.

应当更多地使用内置函数来完成工作.

### 12.不要使用for/while + else的组合

`for/while/else`几乎是所有语言共同的关键字,Python也不例外.

但是Python允许for/while分别和else组合,这在其他语言并不常见.

语法特性比较诡异,如果是掌握多门语言的程序员对此要慎重选择.

### 13.try/except/else/finally

在捕获异常时,同时使用这4种关键字可以达成很强的效果.

应该合理充分利用.


## 小结

本章主要讲了Python一些简单的惯用法:

1. 美观易读非常重要.

2. 充分利用Python的语法特性.如果和第一条发生冲突,优先第一条
