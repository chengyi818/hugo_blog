---
layout: true
title: 把Effective Python读薄之三
categories: 编程语言
tags: [python, effective python]
description: Effective Python 读书笔记 篇三
photos:
- https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1491139399&di=531cd8f9561004e6332e065fdd3a5b98&imgtype=jpg&er=1&src=http%3A%2F%2Fy.zdmimg.com%2F201610%2F01%2F57ef9267469454885.jpg_a200.jpg
---

本文是我总结的Effective Python的第三章

阅读条件:
> 掌握Python类的基本使用

<!--more-->

## 类与继承
Python作为面向对象的一门语言,完整地支持了所有面向对象的特性.继承,多态,封装.

本章作者主要讲了如何利用Python的特性,写出易于维护的代码.

### 22. 辅助类
在维护一个对象的属性时,如果属性比较简单,那么使用字典或者元组之类的内置数据类型比较合适.

在属性变得复杂时,我们需要及时抽象出辅助类,用于管理属性.

### 23. 简单接口接受函数
Python中函数是一级对象(first-class),可以传递.

Python中的某些内置函数,比如sort的参数key可以接受函数.此时应该尽量使用函数.

如果实在有其他需求,可以使用类的实例,同时实现其`__call__`方法

### 24. 以@classmethod形式的多态去通用地构建对象
在Python中一切都是对象,不仅实例是一个对象,类本身也是对象.

可以通过@classmethod来实现类的多态.

### 25. 用super初始化父类

在子类中调用父类的`__init__`方法可能引入一些副作用,所以还是推荐使用super函数来初始化父类.

在Python3中的实现方法如下:`super().__init__()`

### 26. 仅在使用mix-in组件制作工具类时使用多重继承

不管在任何面向对象的使用多重继承都是比较危险的. 在Python中,仅推荐在使用mix-in组件制作工具类时使用.

### 27. 多用public属性,少用private属性

Python的private保护是一种伪实现,仅仅将属性稍加改名.

推荐多使用protect属性,作为一种约定的保护.

只有当子类不受自己控制时,才考虑使用private属性来避免名称冲突

### 28. 使用`collections.abc`来实现自己的容器类型

我们都知道,python的内置容器类型,比如list, set等都实现了大量的内置方法.

如果我们想实现自己的容器,如果要支持默认函数,比如len(),print()等.需要实现大量的方法.

如果继承`collections.adc`则会将必须实现的加以提示,能使用默认提供的则会使用默认提供的方法.

## 小结

本章我们主要讲了写python函数的一些注意事项.
