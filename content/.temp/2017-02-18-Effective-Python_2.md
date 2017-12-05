---
layout: true
title: 把Effective Python读薄之二
categories: 编程语言
tags: [python, effective python]
description: Effective Python 读书笔记 篇二
photos:
- https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1488161464&di=0e6d6512005cb035a797ee94d7b03e8d&imgtype=jpg&er=1&src=http%3A%2F%2Fimg4.duitang.com%2Fuploads%2Fitem%2F201610%2F01%2F20161001182644_kLa2c.jpeg
---

本文是我总结的Effective Python的第二章

阅读条件:
> 掌握Python基本语法

<!--more-->

## 函数

函数几乎是所有语言中的重要组成部分,书中用了一章的内容来介绍更为Pythonic的函数用法.

### 14. 特殊情况
Python支持catch异常,所以在遇到错误情况时,我们应该主动抛出异常,帮助及时发现问题.

而不是返回None或者其他数值,这样会掩盖问题.

### 15. 闭包
首先我们看看关于闭包的内容:
1. 闭包是定义在某个作用域的函数
2. Python中函数是一级对象,即函数可以被引用,可以赋值给变量,可以当做参数传递给其他函数.

其次,我们看看表达式**引用变量**时的遍历顺序:
1. 当前函数的作用域
2. 任何外围作用域(例如, 包含当前函数的其他函数)
3. 包含当前代码的那个模块的作用域(也叫, 全局作用域)
4. 内置作用域(即, 包含len及str等函数的作用域).通常而言,我们不会用到这个作用域
若上述作用域都没有发现同名的变量,Python将会抛出NameError

这也就是我们说的LEGB(local,enclosing,global,built-in)

与之相对的,在给**变量赋值**时,情况大不相同:
1. 若在当前函数作用域已经定义过该变量,则该变量值发生变量.
2. 若尚未定义过该变量,则这次赋值将视为该变量的定义,其作用域也就是当前函数.

综合看下,变量的范围问题,我们可以看到在引用变量时,范围逐级向外扩张.但是在变量赋值时,我们会将作用域限死在当前作用域.
这主要是防止函数中的局部变量污染其他的作用域.

那么,如果我非要在局部函数中修改其他作用域的变量呢?
在Python3中,可以通过`nonlocal`关键字修改外围作用域,通过`global`关键字修改模块的全局作用域.

### 16. 生成器
在前一篇我们讲过关于列表生成器.对于占用小内存而言,列表生成器确实不错.

但是如果数据量较大,那么生成器就是一个更好的选择了.

生成器会在每次迭代后,再进行计算.

### 17. 迭代器
在执行类似`for x in foo`这样的语句时,Python首先会调用`iter(foo)`.

然后,内置函数`iter`又会调用`foo.__iter__`方法.此时该方法必须返回一个迭代器对象.

返回的迭代器对象则必须实现名为`__next__`的方法.

此后,for循环将会在迭代器对象上调用内置的next函数,而该函数则会调用迭代器对象实现的`__next__`方法.

直至迭代器对象抛出`StopIteration`异常.

在使用迭代器时,务必注意每个迭代器仅能迭代一次.

## 参数
在函数的使用过程中,参数的使用和传递必然是很重要的.本章使用了4个小节来讲述的各种参数的使用方法

### 18. 位置参数/星号参数
```
def log(message, *values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print('%s: %s' % (message, values_str))

log('My numbers are', 1, 2)
log('Hi there')
>>>
My numbers are: 1, 2
Hi there
```
可以上面的函数定义中,log函数可以接受不定数量的参数.除了第一个参数,其余参数将会被放到元组`values`中.

星号参数在使用中,需要注意两点:
1. 在星号参数为生成器时,生成器将会转化为元组,这时可能会占用大量内存.
2. 在我们重构log函数时,原先调用log的函数的语句需要重新检查,确保符合我们的预想.

我个人觉得,星号参数不是特别容易控制.尤其是软件开发周期比较长,经历多次重构后.我个人不推荐使用.

### 19. 关键字参数
在定义函数和调用函数时,我们可以以关键字的形式给出参数.
好处:
1. 在调用参数时,明确参数的含义,不容易发生参数误传.
2. 在定义函数时,可以给出参数的默认值.
3. 在重构函数时,很容易和原先的代码兼容.原先调用函数的部分,往往不需要经过修改.

### 20. 动态默认值的参数
```
def log(message, when=None):
    when = datetime.now() if when is None else when
    print("%s: %s" % (when, message))
```

如果在调用参数时,我们希望参数的值是动态获得的.
比如当前的时间戳,那么应当在函数体中获得,而不是通过参数传递.

另外需要注意:
> 参数的默认值,只会在程序加载模块并读到本函数的定义时评估一次.对于{}或[]等动态的值,这可能会导致奇怪的行为.

### 21. 只能以关键字给出参数
```python
def safe_division_c(number, divisor, *,
                    ignore_overflow=False, ignore_zero_division=False):
    ...
```
Python3:
> 在调用上面的函数时,参数`ignore_overflow`和`ignore_zero_divison`将必须以关键字的形式给出.

## 小结
以上是Effective Python的第二章的总结.
