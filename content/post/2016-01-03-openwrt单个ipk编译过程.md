---
title: "openwrt单个ipk编译过程"
tags: ["openwrt","Makefile"]
date: 2016-01-03T19:16:56+08:00
---

本周是成胖子每周一博的第五周.
更好的阅读体验,请点击[这里](https://www.zybuluo.com/icyriver/note/257687)

<!--more-->

---

## 前言
&emsp;前一篇博客中,我们已经知道整个openwrt的编译顺序,本文我们来探讨与开发者息息相关的单个ipk的编译过程.在开发者进行二次开发的时候,我们既可以单个编译ipk也可以完整编译整个镜像文件.在完整编译的时候,我们选中的单个ipk同样会被编入镜像文件中,所以完整编译同样会进行单个ipk包的编译.

我们前面在*stampfile*函数部分提高过,当编译目标为**package/stamp-compile**的时候,实际执行的目标为**package/compile**;同时根据*subdir*函数的定义,**package/compile**将会依赖于package文件夹下被`make menuconfig`选中的子文件夹的**compile**.简而言之,当我们执行`make package/compile`相当于对所有选中的文件夹执行`make package/XXX/compile`.

---
## ipk Makefile分析
我们以一个具体的包的编译过程来看看,本文我们以package/network/services/dropbear这个包为例.当我们在命令行中输入`make package/network/services/dropbear/compile`的时候,**make**将会读入dropbear下的Makefile文件,同时目标指定为**compile**.

因为空间问题,我在这里不展开具体的Makefile文件.相信能看这篇博客的同学应该都有源码,自己打开便是.
下面我们根据**GNU make**语法来分析这个Makefile文件.它包含了两个.mk文件:一个是rules.mk,另一个是package.mk.

rules.mk:
>这个文件我们前文已经提到过了,主要是大量变量的定义.包括各种路径的定义,编译器的定义等等.其中要说明的是**.config**文件也是这里被包含进来的.

package.mk
>这个文件首先定义和补充了一些变量.其次是**openwrt**为我们封装了**BuildPackage**函数,对于普通开发者而言,只需要参照模板定义相应的变量,最后调用这个函数即可.

其余的我们可以认为是变量的赋值语句,很明显使用它们的地方并不在这里.关于模板和变量值的说明及作用.,你可以参照[官方说明](https://wiki.openwrt.org/doc/devel/packages),也可以在网上找到一大堆资料.

最后,最重要的语句是这一句:

```
$(eval $(call BuildPackage,dropbear))
```

这里将会把**dropbear**作为参数值传给函数**BuildPackage**

Tips
:   不知道大家还记得我们Makefile的执行顺序么?Makefile是先读入所有信息,展开,然后生成依赖关系.最后再按依赖关系先后来执行.

---
## 依赖关系

### BuildPackage分析
**BuildPackage**的定义在**package.mk**中,定义如下:

```
define BuildPackage
  $(Build/IncludeOverlay)
  $(eval $(Package/Default))
  $(eval $(Package/$(1)))

ifdef DESCRIPTION
$$(error DESCRIPTION:= is obsolete, use Package/PKG_NAME/description)
endif

ifndef Package/$(1)/description
define Package/$(1)/description
	$(TITLE)
endef
endif

  BUILD_PACKAGES += $(1)
  $(STAMP_PREPARED): $$(if $(QUILT)$(DUMP),,$(call find_library_dependencies,$(DEPENDS)))

  $(foreach FIELD, TITLE CATEGORY SECTION VERSION,
    ifeq ($($(FIELD)),)
      $$(error Package/$(1) is missing the $(FIELD) field)
    endif
  )

  $(if $(DUMP), \
    $(Dumpinfo/Package), \
    $(foreach target, \
      $(if $(Package/$(1)/targets),$(Package/$(1)/targets), \
        $(if $(PKG_TARGETS),$(PKG_TARGETS), ipkg) \
      ), $(BuildTarget/$(target)) \
    ) \
  )
  $(if $(PKG_HOST_ONLY)$(DUMP),,$(call Build/DefaultTargets,$(1)))
endef
```

那么这里的$(1)就是指的传入的参数**dropbear**.这里包含了一些检查和补充变量定义.继续深究下去的线索是第25~32行之间.这里我将它简化后就是展开**BuildTarget/ipkg**;同时第33行,将dropbear当作参数传给函数**Build/DefaultTargets**.

**BuildTarget/ipkg**定义在**package-ipkg.mk**中,我们需要重点关注其中的冒号,这个形成我们的依赖关系.
**Build/DefaultTargets**定义在**package.mk**中,其中形成了我们stamp-*的依赖关系.根据这些依赖关系,我将关系图绘制如下:
![package/compile](http://ww4.sinaimg.cn/large/006kvZhRjw1ezmd5eo1iej31bb0fcwh0.jpg)

## 执行
当我们得出依赖关系后,执行过程就是倒序进行而已,即从上图的右边向左执行.这也可以和我们预料的执行过程相印证.

$(stamp-prepared)
:   主要完成代码包的准备工作,如果开发者定义了**build/prepare**,则执行**build/prepare**.如果开发者未定义,则执行**build/prepare/default**这其中包含了多个情形,最为常见的是将**dl**下的压缩包解压并打上patch.

$(stamp-built)
>这个将会进入到**build_dir/target-XXX/**下对应的文件夹进行编译.同时将会带入一些定义好的变量.比如CFLAGS,LDFLAGS.

IPKG_(1)
>这个目标将会将编译好的文件安装到对应的**ipkg-arch**目录下,同时将这个目录打包为ipk文件.

---
## 尾记

本周博客基本就到这里,本来私心想着元旦没啥大事,可以写两篇的.结果混着混着就到第三天晚上了.剩下的最后一篇我们看看单个的ipk编译好了,内核的编译过程,最后的打包过程.整个镜像文件由哪些部分组成.下周再见.
