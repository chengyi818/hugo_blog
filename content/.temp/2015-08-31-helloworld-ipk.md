---
layout: post
title: openwrt简单ipk生成及Makefile解释
categories: openwrt
tags: openwrt
comments: true
analytics: true
---



##前言

类似的文章其实网上比较多了,我写这个的目的:
>1,网上文章良莠不齐,有些自己都没实际动手操作,随便复制粘贴,实际操作不可行.
2,基本只讲了操作,我当时最关心的Makefile文件的解释没有.

所以我自己总结了一篇.

<!--more-->

---

###说明
开发板为MT7620a,openwrt版本为:barrier_breaker_14.07.编译主机为ubuntu 14.04 32位.
`git clone git://git.openwrt.org/14.07/openwrt.git`
关于怎么搭建编译环境以及编译请参考[网上](http://blog.chinaunix.net/uid-22547469-id-4364254.html)

---

##正文
下面我们开始,我们遵循传统以helloworld开始.
### 1.创建helloworld项目
首先我们新建helloworld.c文件和对应的Makefile文件

```
$mkdir -p ~/temp/hellworld/src
$cd ~/temp/helloworld/src
$touch helloworld.c Makefile
```

如下为helloworld.c的内容:

```
#include <stdio.h>
int main()
{
    printf("This is my helloworld!\n");
    return 0;
}
```

如下为Makefile文件的内容:

```
helloworld : helloworld.o
	$(CC) $(LDFLAGS) helloworld.o -o helloworld

helloworld.o : helloworld.c
	$(CC) $(CFLAGS) -c helloworld.c

clean :
	rm *.o helloworld
```

$(CC)
>   这个值由其他Makefile文件规定,表示我们使用编译器.

$(LDFLAGS)\$(CFLAGS)
>   这个表示编译器的一些选项,这里是可选的,去掉也没有问题.

下面可以输入`$make`看看有没有问题,注意Makefile文件的书写格式.
最后,输入`$make clean`来清理掉生成的二进制文件.因为上一步`make`所使用的编译器并不是我们的交叉编译链,生成的二进制文件并不能在开发板中运行.上一步只是验证我们的src中的内容正确与否.

---
### 2.创建helloworld包
下一步我们要创建一个新的Makefile文件,在这个文件中我们要描述的是helloworld包的信息,比如:如何配置,如何编译,如何打包,安装位置等.

```
$cd ~/temp/helloworld
$touch Makefile
```

如下为Makefile内容:

```
include $(TOPDIR)/rules.mk

PKG_NAME:=helloworld
PKG_RELEASE:=1
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)

include $(INCLUDE_DIR)/package.mk

define Package/helloworld
	SECTION:=utils
	CATEGORY:=Utilities
	TITLE:=Helloworld -- prints a snarky message
endef

define Package/helloworld/description
	It's my first package demo.
endef

define Build/Prepare
	echo "Here is Package/Prepare"
	mkdir -p $(PKG_BUILD_DIR)
    $(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Package/helloworld/install
	echo "Here is Package/install"
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
endef

$(eval $(call BuildPackage,helloworld))
```

如下是最后的文件树形图:

![树形图](http://i3.tietuku.com/8e9f244567388099.jpg)

---

### 3.Makefile注释

第1行`include $(TOPDIR)/rules.mk`
>   一般在Makefile的开头,包含了包的基本信息,

>比如Makefile中的`$(BUILD_DIR)`,`$(INCLUDE_DIR)`,`$(CP)`,`$(INSTALL_DIR)`,`$(INSTALL_BIN)`都是这里定义的.具体内容可以到源码主目录下,查看`rules.mk`文件.

3~5行,软件包的信息均以“PKG_”开头，其意思和作用如下
>   PKG_NAME：软件包名称，将在menuconfig和ipkg可以看到。

>    PKG_VERSION：软件版本号。

>    PKG_RELEASE：Makefile的版本号

>    PKG_SOURCE：源代码的文件名。

>    PKG_SOURCE_URL：源代码的下载网站位置。

>    PKG_MD5SUM：源代码文件的效验码。用于核对软件包是否下载正确。

>    PKG_CAT：源代码文件的解压方法。包括zcat, bzcat, unzip等。

>    PKG_BUILD_DIR：软件包编译目录。它的父目录为$(BUILD_DIR)。

第7行`include $(INCLUDE_DIR)/package.mk`
>   一般在软件包的基本信息完成后再引入，他定义了用户态软件包的规则。

>编译包分为用户态和内核模块，用户态软件包使用Package，内核模块使用KernelPackage.

>`$(INCLUDE_DIR)/Kernel.mk`文件对于软件包为内核时不可缺少，

>`$(INCLUDE_DIR)/package.mk`应用在用户态。

>接下来讲述用户态软件包。用户程序的编译包以`Package/`开头，然后接着软件名，在Package定义中的软件名可以与软件包名不一样，而且可以多个定义。

9~13行
>   定义包的名称为`helloworld`

> SECTION : 包的类型为`utils`

> CATEGORY : 目录为Utilitis,即文件在`menuconfig`中的位置;有时还会有`SUBMENU`项,即子目录.

> TITLE : 用于软件包的简短描述,将显示在`menuconfig`中.

> URL ： 软件包的下载位置。

> MAINTAINER ： 维护者选项。

> **DEPENDS** ： 与其他软件的依赖。即如编译或安装需要其他软件时需要说明。如果存在多个依赖，则每个依赖需用空格分开。依赖前使用+号表示默认显示，即对象沒有选中时也会显示，使用@则默认为不显示，即当依赖对象选中后才显示。

15~17行
>   软件包的详细描述,将显示在`make menuconfig`中

19~23行
>   编译准备方法，对于网上下载的软件包不需要再描述。对于非网上下载或自行开发的软件包必须说明编译准备方法。

> 本文所用的准备方法就是首先创建软件包目录，然后将源码拷贝到刚刚创建的目录中。按OpenWrt的习惯，一般把自己设计的程序全部放在src目录下。

25~29行
>   软件包的安装方法，包括一系列拷贝编译好的文件到指定位置。调用时会带一个参数，就是嵌入系统的镜像文件系统目录，因此$(1)表示嵌入系统的镜像目录。

>`INSTALL_DIR:=install -d -m0755` : 创建所属用戶可读写、执行，其他用戶可读可执行的目录

>`INSTALL_BIN:=install -m0755` : 编译好的文件到镜像文件目录

31行 `$(eval $(call BuildPackage,helloworld))`
>   完成前面定义后，必须使用eval函数实现各种定义。其格式为：

> 对于一般软件包：`$(eval $(call Package,$(PKG_NAME)))`

> 或对于内核模块：`$(eval $(call KernelPackage,$(PKG_NAME)))`

> 如果一个软件包有多个程序，例如：一个应用程序有自己的内核模块，上面使用的`PKG_NAME`需要灵活变通。`eval`函数可能设计多个。也可以当成多个软件包处理。

这里简单地解释了Makefile文件,更具体地请[参考](http://wiki.openwrt.org/doc/devel/packages)

---
### 4.编译软件
至此我们的软件已经基本完成,下面进行编译
首先将文件文件夹拷贝到**openwrt目录中的package文件中**,这里我的源码目录为`~/openwrt`,你需要把openwrt目录替换为你的openwrt源码目录.

```
$mv ~/temp/helloworld ~/openwrt/package
```

然后回到项目主目录运行make menuconfig

```
$cd ~/openwrt
$make menuconfig
```

按"/"后,输入helloworld,搜索对应的路径

![搜索](http://i3.tietuku.com/3376b2fa48621a57.png)
![搜索结果](http://i3.tietuku.com/9947195c15c0d77f.png)

接着到Utilities目录下,找到helloworld并按空格打开;
![打开编译开关](http://i3.tietuku.com/b389aec1d8d367ee.png)

保存后退出;

```
$cd ~/openwrt
$make package/helloworld/compile V=s
```

编译完成后,ipk应该已经生成

```
$find bin/ -name "helloworld*.ipk"
```

至此我们已经生成简单的ipk,恭喜:)
最后可以通过[winscp](http://pan.baidu.com/s/1bnHfXyJ),将ipk[安装](http://www.openwrt.org.cn/bbs/forum.php?mod=viewthread&tid=3238)到开发板中.

![结局](http://i3.tietuku.com/1ac2f9939aa02cb9.jpg)

---
##尾记
我比较薄弱的是Makefile方面的知识,刚好加强下这个方面的学习,欢迎交流~
