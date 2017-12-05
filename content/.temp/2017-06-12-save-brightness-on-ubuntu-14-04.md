---
layout: true
title: Ubuntu 14.04保存屏幕亮度
tags: [linux]
photos:
  - http://www.ctdf.org.tw/upload/times/20160524141100.jpg
date: 2017-06-12 17:24:22
categories: 工具
description: 如何在Ubuntu 14.04上保存屏幕亮度的设置
---

阅读条件:
> Ubuntu 14.04使用者

---

<!--more-->

## 前言
我最近在使用Ubuntu 14.04的过程发现屏幕亮度的设置不能保存.默认情况下屏幕一直保持最高亮度.

在网上找了些链接加上自己的经验,仅供参考.

## 参考链接
[参考链接](https://my.oschina.net/SBaof/blog/479439)大体都是类似的.

主要思路是既然不能保存屏幕亮度,那么我们就利用启动脚本来帮我们在每次开机的时候,主动设置亮度.

总体来说,这个思路是没问题的.但是链接上一般给出的设备是`acpi_video0`.

```
$ sudo ls /sys/class/backlight
acpi_video0 intel_backlight
```
在我的电脑上使用的是英特尔的核心显卡,所以对应的目录应该是`intel_backlight`.其他的思路是类似的.

## 解决方案

1. 首先需要确定你的显卡使用的是什么?如果觉得麻烦,就把`backlight`下的目录都写到`rc.local`中.
2. 其他思路参照[参考链接](https://my.oschina.net/SBaof/blog/479439)即可
3. 注意: 不同目录对应的最大亮度数值是不同.
