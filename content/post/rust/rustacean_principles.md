---
title: "rust编程原则"
date: 2021-09-12T19:49:35+08:00
draft: true
lastmod: 2021-09-12T19:49:35+08:00
tags: ["rust"]
categories: ["rust****

---

# 前言
rust作为目前比较受欢迎的新兴语言,能够比较好的兼顾安全性和性能. 我们通过*参考资料1*来了解rust社区发展和开发rust本身的一些原则. 这些原则分为两个部分,一部分是rust语言设计追求的原则,另一部分则是如何成为一个更好的rust社区成员.

通过学习rust的设计理念,对于深入掌握rust语言也有所助益.

# Rust的设计目标
rust希望赋予开发者构建可靠高效软件的能力.在不冲突的情况下,当然希望可以同时满足以下所有设计目标.在不得不取舍时,从上到下,优先级依次递减.

## Reliable
> if it compiles, it works
rust希望当程序通过编译后,就可以正确地运行. 这样你可以做较大范围的重构,而不用担心正确性. 类似地,当你使用并发编程或者其他高级特性时,当通过编译后程序就应该正确地运行, 而不是需要各种fuzz测试或者静态动态检查来保证程序的正确性.

C/C++类的语言由于天然的内存不安全特性,在大型项目开发过程中,往往需要大量的额外检查来保证其正确性,这其实带来了相当大的负担.而类似Python这样的解释型语言在运行前,则根本无法保证其正确性.rust将检查和规则一部分前移到了编译期,另一部分则通过和开发者的约定来完成.这带来的一个后果就是开发者的学习成本和第一次开发成本会有增加,但是长期来看,心智负担和维护成本则更小.

回到语言层面,*类型安全*和*强制考虑所有场景*是保证可靠性的主要手段.

## Performant
> idiomatic code runs efficiently
rust语言特性经过了精心地优化,一些语言特性如闭包,迭代器等,既运行迅速占用内存也很少.

目前来看,rust提供了和C/C++相当的运行效率. 类似于C++, rust也尽量遵循零成本抽象原则,在汇编层面并未增加过多运行负担.

## Supportive
> the language, tools, and community are here to help
语言本身,相关工具和社区都竭尽全力对开发者更加友好.

这一点从rust相关的文档社区,甚至编译器的细节都能感受到. rust在编译时,相较于其他语言确实会稍微困难些,但是其编译提示真的是非常nice.

## Productive
> a little effort does a lot of work
rust希望通过提供大量强大的可复用组件来赋予开发者快速构建复杂系统的能力.

rust语言本身提供的crate机制非常好用,类似于python的pip. 早期设计的语言如C/C++在设计之初没有考虑到这一点,导致需要通过后期的工具去弥补.从开箱即用的角度来看,确实明显不如这些00后的语言.

## Transparent
> you can predict and control low-level details
从rust到汇编层面这样的底层机器代码转换是很直接的.同时语言层面提供了直接控制底层细节的能力,这就为rust作为系统级语言提供了能力.

C语言从语言到汇编就很直接,但是C++在编译器中干了太多的魔法,导致就不是很直接.坦白讲,你就不得不学习很多语言层面的约定,加重了额外的心智负担.

## Versatile
> you can do anything with rust
rust拥有广泛的适应性.从最简单的脚本到web服务器,WebAssembly再到内核扩展模块和Os,应该说虽然rust目前还没有很有统治力的杀手级应用,但是它的应用范围是非常广阔的.

目前python,java等虽然很火热,但是由于其天然的设计缺陷,一些特定领域它们是注定无法进入的.

# 如何成为更好的rust社区成员

## Be kind and considerate

## Bring joy to the user

## Show up

## Recognize other's knowledge

## Start somewhere

## Follow through

## Pay it forward

## Trust and delegate

# 案例
通过实际案例了解以上介绍的内容是如何影响rust的设计与实现.

# 小结
我个人感觉通过本文去了解rust的设计理念和社区文化, 从而去了解rust背后的理念. 本文主要是摘录重点, 随便写了些自己的看法. 更详细的内容请移步*参考资料3*

# 参考资料
1. [Rustacean Principles](https://smallcultfollowing.com/babysteps//blog/2021/09/08/rustacean-principles//)
2. [Rustacean Principles website](https://rustacean-principles.netlify.app/what_is_rust.html)
3. [Rustacean Principles github repo](https://github.com/nikomatsakis/rustacean-principles)
