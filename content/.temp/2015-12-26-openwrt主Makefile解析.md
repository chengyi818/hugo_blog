---
layout: post
title: openwrt主Makefile解析
categories: openwrt
tags: [makefile,openwrt,深度]
comments: true
analytics: true
---

本周成胖子每周一博到了第四周

## 前言
前一篇,我们大概描述了整个镜像文件的生成过程.本周我们来解析主Makefile,看看主要编译过程是怎么产生的.

<!--more-->

## 主Makefile结构
我们以`chaos calmer`的代码为例,整个编译的入口是在源码根目录下的Makefile.编译的各种命令都应该在源码根目录下键入.
整个主Makefile的结构如下:

```
world:
ifneq ($(OPENWRT_BUILD),1)
	顶层
else
	第二层
endif
```

开始部分是一些注释和变量定义及路径检查.
根据**Makefile的规则**,在没有指定编译目标的时候,Makefile中的第一个目标将作为默认目标.
换句话说,当我们执行`make V=s`时,这个时候编译的目标就是`world`.和我们执行`make world V=s`效果是一样的.

## 顶层
通常在编译时,我们不会定义变量`OPENWRT_BUILD`的值,所以通常我们是会走到顶层的.
顶层代码如下:

```bash
  _SINGLE=export MAKEFLAGS=$(space);

  override OPENWRT_BUILD=1
  export OPENWRT_BUILD
  GREP_OPTIONS=
  export GREP_OPTIONS
  include $(TOPDIR)/include/debug.mk
  include $(TOPDIR)/include/depends.mk
  include $(TOPDIR)/include/toplevel.mk
```

这里我们看到变量`OPENWRT_BUILD`被置为1.然后包含了3个`.mk`文件.
这里稍微解释下`.mk`文件.它们一般没有什么执行动作,都是一些变量的定义还有依赖关系的说明.可以类比于C语言的头文件来理解.

debug.mk:
> 可以通过定义DEBUG的值来控制编译过程

depends.mk
> 主要定义了rdep这个变量

toplevel.mk
>这个是我们跟踪编译过程的重要的文件.这个文件在源码根目录下的`include`文件夹下.

核心代码如下:

```
%::
	@+$(PREP_MK) $(NO_TRACE_MAKE) -r -s prereq
	@( \
		cp .config tmp/.config; \
		./scripts/config/conf --defconfig=tmp/.config -w tmp/.config Config.in > /dev/null 2>&1; \
		if ./scripts/kconfig.pl '>' .config tmp/.config | grep -q CONFIG; then \
			printf "$(_R)WARNING: your configuration is out of sync. Please run make menuconfig, oldconfig or defconfig!$(_N)\n" >&2; \
		fi \
	)
	@+$(ULIMIT_FIX) $(SUBMAKE) -r $@ $(if $(WARN_PARALLEL_ERROR), || { \
		printf "$(_R)Build failed - please re-run with -j1 to see the real error message$(_N)\n" >&2; \
		false; \
	} )
```

除了少数在toplevel中被定义的目标外,其他编译目标都会走到这里.将之简化后:

```
%::
	make prereq
	make $@
```

首先执行`prereq`,然后再执行我们指定的目标或者默认目标`world`.
prereq整理后的依赖关系如下:
![prereq](http://img.blog.csdn.net/20151226142730074)
其中
staging_dir/host/.prereq-build:
> 将会执行一系列主机检查,是否安装了必要的软件.

prepare-tmpinfo:
> 根据scan.mk,扫描`target/linux`和`package`目录,生成packageinfo和targetinfo.

总之,顶层完成一系列必要的准备工作.对于绝大多数的目标而言,顶层是必经之路.当然,在`toplevel.mk`中,我们也可以看到目标`menuconfig`.也就是说对于目标`menuconfig`而言,将不会执行到第二层的逻辑.


## 第二层
在上面执行完`make prereq`之后,将执行`make world`.
还记得我们进入顶层后修改了变量`OPENWRT_BUILD`么?当再次执行`make world`的时候,由于条件不满足,我们将直接进入第二层来执行.

```
  include rules.mk
  include $(INCLUDE_DIR)/depends.mk
  include $(INCLUDE_DIR)/subdir.mk
  include target/Makefile
  include package/Makefile
  include tools/Makefile
  include toolchain/Makefile
```

rules.mk:
> 很重要的一个mk文件,其中规定了很多有用的变量,包括各种目录路径的定义,交叉编译器等等.其中
> ```
> ifeq ($(DUMP),)
  -include $(TOPDIR)/.config
endif
> ```
> 就是包含了我们的配置文件.对于`Makefile`而言,`.config`文件就是一大串变量的定义.Makefile可以直接读取这些定义,从而控制编译过程.

subdir.mk:
> 这个是读懂我们整个编译过程的**关键**所在,其中主要定义了两个函数:*subdir*和*stampfile*,我们稍后加以解释.

接下来,包含了4个Makefile文件.我们以`target/Makefile`为例.该文件位于`target`目录下.
核心部分为:

```
$(eval $(call stampfile,$(curdir),target,prereq,.config))
$(eval $(call stampfile,$(curdir),target,compile,$(TMP_DIR)/.build))
$(eval $(call stampfile,$(curdir),target,install,$(TMP_DIR)/.build))

$(eval $(call subdir,$(curdir)))
```

这里调用了`subdir.mk`中定义的`stampfile`函数.将会生成`target/stamp-prereq`,`target/stamp-compile`,`target/stamp-install`三个变量.
以`target/stamp-prereq`为例,执行部分为`make target/prereq`.同理`target/stamp-compile`,执行部分为`make target/compile`.

最后又调用了`sbudir`函数,这个函数规定了目标和各子文件夹之间的依赖关系.如果有一定的Makefile基础可以去读读`subdir.mk`文件.
举例而言就是:
> 当执行目标为`target/compile`,这个目标将依赖于`target/linux/compile`.
> 当执行目标为`package/compile`,这个目标将依赖于`package`目录下,各子文件夹的`compile`.

下面就是规定了一系列的依赖关系,我已经将他们梳理了出来,如下图:
![world](http://img.blog.csdn.net/20151226151210514)

这里就是第二层解析后的依赖关系.当依赖关系生成后,将会从最先被依赖的目标开始执行.
比如我们可以看到进入第二层后,`tools/stamp-install`将会最先被执行,也就是主机工具将会最先被编译,安装.我们上一篇提高的整个编译过程能从上图中得出.


## 尾记
1. 想要读懂Makefile,首先要梳理各个依赖关系.而要梳理各个依赖关系,关键要关注冒号和`make -C`
2. 本周我们解析了主Makefile,在Makefile的执行过程中要理解make的执行过程.先读入Makefile,然后构建依赖关系,最后最先被依赖的目标将会先执行.
3. 我主要描绘了主要枝干,如果希望了解更多细节,还是要自己去阅读Makefile.
3. 接下来两篇,我们将主要分析下,和我们开发者比较相关的两个目标的执行过程:`package/stamp-compile`和`target/stamp-install`.下周再会^_^
