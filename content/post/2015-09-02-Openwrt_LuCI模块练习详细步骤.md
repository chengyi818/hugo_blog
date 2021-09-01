---
title: "Openwrt LuCI模块练习详细步骤"
tags: ["openwrt"]
date: 2015-09-02T19:16:56+08:00
---


## 前言
又到了成胖子每周一博的时间了.最近在学习openwrt luci方面的知识,为了贯穿整个知识体系,练习题目为:

> 通过页面配置周期性地往/tmp/addtest文件写入内容和时间戳

> 1.在web主页面的下拉菜单做一个按钮,进入设置页面;

> 2.两个设置项:输入的内容和周期;

> 3.读取/tmp/addtest中的内容并显示在页面上;

代码已经[开源](https://github.com/chengyi818/addtest),欢迎交流~

<!--more-->

---

## 知识准备

### 源码编译及ipk生成
这部分网上相关文章很多,也可以参见[拙作](http://www.cnblogs.com/chengyi818/p/4774043.html)

### LuCI
首先回答一个问题:什么是Luci?

~~>LuCI是OpenWrt上的Web管理界面，LuCI采用了MVC三层架构，使用Lua脚本开发.~~

>简单地说,Luci就是用来做openwrt的页面的.不同于常见的html+css+javascript,Openwrt是用lua脚本语言开发的.

怎么开发一个页面呢?

>要开发一个新的功能页面,开发者只要根据MVC框架写些简单的lua脚本,剩下的部分由openwrt为你自动完成.

说到MVC框架了,什么是MVC框架呢?

>MVC是model+view+controller的简写.为了便于开发,openwrt将实现不同功能的lua脚本放在不同的文件夹中.请看下图:

![MVC架构](http://i3.tietuku.com/11e2965d6e4c95e4.jpg)

什么是controller控制器?

>我们在这里设置功能在页面的位置,同时设置点击页面后,将要调用的功能.是要去Model模型读写配置数据呢?还是要呈现一个静态页面,或者是直接执行lua脚本函数.

什么是model模型?

>这里我们常用的是,通过cbi模块和UCI(统一配置接口)进行交互.简单地说,就是我们在这里将页面和路由器里面的配置关联起来,从而将页面的设置写到路由器当中.

什么是view视图?

>这个应该是最容易理解的,就是呈现的页面的样式,有点类似于传统的html页面.

上面说到了UCI(Unified Configuartion Interface),这是什么龟?

>openwrt将配置用统一的格式书写,放在规定的地方(/etc/config/),同时提供接口函数进行读取和设置.

如果还不太明白,接着向下看.如果有可能跟着我动动手,相信你很快就会掌握:)

---

## 正文
我们先看下最终效果图:

![最终效果](http://i3.tietuku.com/cf83e41dbd8859bf.jpg)

我们在页面上面的`System`下拉框的下面加了一个`AddTest`按钮,下面有两个子选项:`Set`和`Info`.其中`Set`用于选择是否开启功能,设置时间间隔和内容.`Info`用于显示`/tmp/addtest`文件中的内容.

### 准备工作
首先,嗯~
你得有环境,得有电,有源码,编译过简单的ipk.如果没有,请[回炉](http://www.cnblogs.com/chengyi818/p/4774043.html)重造.

其次,建立相应的文件夹及文件.至于linux操作神马的,我相信你一定没有问题.

```
$mkdir -p ~/temp/addtest
$cd ~/temp/addtest
```

最终文件树形图

![树形图](http://i3.tietuku.com/e0e8a0b049ed0f45.jpg)

骨架已经有了,下面只需要往里面填肉了,是不是感觉很快~
不要管为什么要这样,我们后面慢慢解释.

---

### controller
前面我们提到,controller主要用于控制页面按钮位置,以及调用的功能.首先来编辑这个文件.

```
$vim ~/temp/addtest/files/usr/lib/lua/luci/controller/addtest.lua
```

代码如下:

```
module("luci.controller.addtest",package.seeall)

function index()
    entry({"admin","system","addtest"},alias("admin","system","addtest","set"),_("AddTest"),99).index=true
    entry({"admin","system","addtest","set"},cbi("addtest"),_("Set"),1)
    entry({"admin","system","addtest","info"},call("action_info"),_("Info"),2)
end

function action_info()
    if not nixio.fs.access("/tmp/addtest") then
        return
    end

    local info = nixio.fs.readfile("/tmp/addtest")
    luci.template.render("addtest_info",{info=info})
end
```

格式模板:

```
module("luci.controller.控制器名", package.seeall)

function index()
        entry(路径, 调用目标, _("显示名称"), 显示顺序)
        end
```

这个脚本文件可以分为3块:第1行,3~7行,9~16行

第1行

> 说明了模块的名称,本文在controller目录下创建了`addtest.lua`文件,将模板中的控制器名替换为`addtest`即可.

第3行

>  第3~7行定义按钮的位置,调用的功能,显示名称.其中第3行和第7行是固定的模板格式,不需要修改

第4行

>   entry表示添加新的模块.
第一个参数`{"admin","system","addtest"}`表示按钮的位置.`admin`表示我们这个功能只有以管理员身份登录页面才可以看到.`system`表示一级菜单名,`addtest`则是一级菜单下的子菜单.第二个参数`alias("admin","system","addtest","set")`表示调用的功能.这个按钮没有独立的功能,而是将它关联到它的下一级子菜单`set`.
第三个参数`_("AddTest")`表示显示名称,可选.如果页面按钮想做成中文,可以在这里设置.
第四个参数`99`表示显示顺序的优先级,Luci根据这个值为同一父菜单的所有子菜单排序.

第5行

> 第一个参数`{"admin","system","addtest","set"}`表示在`addtest`下再增加一个子选项`set`.
第二个参数`cbi("addtest")`表示调用cbi模块,这里将会调用到`/usr/lib/lua/luci/model/cbi/addtest.lua`

第6行

> 第二个参数`call("action_info")`表示执行指定方法,这里将会调用我们下面写的`acttion_info`函数.

备注
:   关于`entry`第二个参数调用目标.我们还有一个`template`没有涉及,它表示访问指定页面.比如`template(addtest_info)`将会直接访问`/usr/lib/lua/luci/view/addtest_info.htm`.

9~16行

>  这里使用lua语言调用`nixio`接口写了一个简单的函数,首先判断文件是否存在,然后读取其中的内容赋值给变量`info`,最后访问指定页面`/usr/lib/lua/luci/view/addtest_info.htm`,同时将变量`info`传递过去.
[luci接口手册](http://luci.subsignal.org/api/luci/)
[nixio接口手册](http://luci.subsignal.org/api/nixio/)

---

### UCI
UCI是openwrt的配置管理机制,它将配置统一放到`/etc/config`文件夹下.详细地介绍请参考[这里](http://www.leiphone.com/news/201406/diy-a-smart-router-topic-system-configuration.html).
下面来编辑这个文件

```
$vim ~/temp/addtest/files/etc/config/addtest
```

代码如下:

```
config arguments
    option interval ''
    option content ''
```

Section开始语法: `config '类型' '名字'`
参数定义语法: `option '键' '值'`
列表定义语法: `list '集合名字' '值'`

简单解释下,我们在`/etc/config`下新建一个名为`addtest`的配置文件,其中类型为`arguments`,名字省略.有两个键,一个名为`interval`用来存时间间隔.一个名为`content`用来存准备周期性输入的内容.

---

### Model
在**controller**章节中,我们提到`cbi`会调用到`model`文件夹中的`addtest.lua`文件.下面我们来编辑它.

```
$vim ~/temp/addtest/files/usr/lib/lua/luci/model/cbi/addtest.lua
```

代码如下:

```
m=Map("addtest",translate("Luci practice"),translate("fat cheng's test"))

s=m:section(TypedSection,"arguments","")
s.addremove=true
s.anonymous=false

s:option(Flag,"enable",translate("Enable"))
s:option(Value,"interval",translate("Interval"))
s:option(Value,"content",translate("Content"))

local apply=luci.http.formvalue("cbi.apply")
if apply then
    io.popen("/etc/init.d/addtestd restart")
end

return m
```

下面我们来解释下这个文件.

第1行

>   模板`m = Map("配置文件文件名", "配置页面标题", "配置页面说明")`
第一个参数:上一步我们新建配置文件`/etc/config/addtest`.这里就是建立与配置文件的联系.
第二,三两个参数,则是页面的主标题和副标题.还不清楚的话,翻上去看看最终效果图,看看它们在哪里.

第3行

>   在一个配置文件中可能有很多Section,所以我们需要创建与配置文件中我们想要的Section的联系.
有两种方式可以选择:NamedSection(name,type,title,description)和TypedSection(type,title,description),前者根据配置文件中的Section名，而后者根据配置文件中的Section类型.我们选用了第二种.

第4行

>   设定不允许增加或删除Section

第5行

>   设定显示Section的名称,这里建议你可以试试设定为`true`,看看会发生什么.

7~9行

>   接着则是建立与Section中的option之间的联系.模板`s:option(交互形式,option键值,显示名称)`.
第一个参数:常见的交互形式有Value(文本框),ListValue(下拉框),Flag(选择框).,不知道为啥我打不开[官方文档](http://luci.subsignal.org/trac/wiki/Documentation/CBI),这里也可以[参考](http://blog.csdn.net/qq_21949217/article/details/44151595)
第二个参数表示在配置文件中的option的键值
第三个参数表示,你希望在页面上呈现的名称.
创建后开发者无需考虑读取以及写入配置文件的问题，系统会自动处理.

11~14行

>   系统会为我们在页面上自动创建一些按钮`Save&Apply`,`Save`,`Reset`.我们仅仅将配置写入`/etc/config`下对应的文件是不够的,我们还希望可以根据这个配置进行一些操作.
这部分代码的作用是,当你按下页面的`apply`按钮后,相当于在串口shell下输入`/etc/init.d/addtestd restart`

---

### init.d
上一节我们已经可以读写配置了,怎么根据配置来进行操作呢?这是我们这一节要谈的.我们来编辑`~/temp/addtest/files/etc/init.d/addtestd`这个文件.
代码如下:

```
#!/bin/sh /etc/rc.common
START=50

run_addtest()
{
    local enable
    config_get_bool enable $1 enable

    if [ $enable ]; then
        local interval
        local content
        config_get interval $1 interval
        config_get content $1 content

        addtest $interval $content
    fi
}

start()
{
    config_load addtest
    config_foreach run_addtest arguments
}

stop()
{
    result=`pidof addtest`
    kill -9 $result
    echo "addtest has stoped"
}
```

第1行

>   Linux 系统根据 "#!" 及该字串后面的信息确定该文件的类型,表示这个文件需要由/bin/sh和/etc/rc.common来解释执行.

第2行

>   表示启动的优先级,这里暂时用不到

4~17行

>   是一个函数,主要作用是读取`/etc/config/addtest`中的内容,然后根据是否打开开关在第15行将配置传递给可执行文件`addtest`,由它根据配置执行指定的操作.
读取配置的方法,我强烈推荐你阅读[官方文档](http://wiki.openwrt.org/doc/devel/config-scripting),精炼而简洁.
获取布尔值类型:`config_get_bool 变量名 Section名 Section参数名`
获取变量值:`config_get 变量名 Section名 Section参数名`

19~23行

>   对应于`/etc/init.d/addtestd start`.首先使用`config_load 配置文件名`的方法载入配置文件,然后使用`config_foreach 遍历函数名 Section类型`的方法,遍历配置文件中的Section.

25~30行

>   对应于`/etc/init.d/addtestd stop`.找到`addtest`这个进程的进程号,然后杀死它

备注
:   前一节提到的`/etc/init.d/addtestd restart`中的`restart`命令,在`/etc/rc.common`进行了定义,简单来讲就是先执行了`stop`命令,再执行`start`命令.
最后务必执行**`$sudo chmod 755 ~/temp/addtest/files/etc/init.d/addtestd`**.

---

### src
前一节,我们谈到`run_addtest`调用可执行文件`addtest`,现在我们编辑这部分内容

```
$vim ~/temp/addtest/files/src/addtest.c
```

代码如下:

```
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    int index;
    for(index=0; index<10; index++)
    {
        FILE *fp=fopen("/tmp/addtest","at");
        system("date >> /tmp/addtest");
        fprintf(fp, "%s\n", argv[2]);
        fclose(fp);
        printf("interval=%d\n",atoi(argv[1]));
        sleep(  atoi(argv[1]) );
    }
    return 0;
}
```

这部分代码比较简短,我们不再解释.需要掌握的点有:

> 1.`argc`和`argv[]`的使用方法

> 2.`fopen`函数,`fclose`函数以及`fprintf`函数的使用方法

> 3.`system`函数的使用方法

> 4.`sleep`函数和`atoi`函数的使用方法,`argv[1]`的类型为`char`需要转换为整型.

通过这个可执行文件,我们周期性地将时间戳和内容写入了`/tmp/addtest`文件.
最后我们写一个简单的Makefile:

```
$vim $vim ~/temp/addtest/files/src/Makefile
```

代码如下:

```
addtest : addtest.o
	$(CC) addtest.o -o addtest

addtest.o : addtest.c
	$(CC) -c addtest.c

clean :
	rm *.o addtest
```

---

### View
上一节,我们已经根据配置将指定的内容周期性地写入了`/tmp/addtest`.在**controller**那一节,我们的函数`action_info`读取了`/tmp/addtest`中的内容并访问指定页面`/usr/lib/lua/luci/view/addtest_info.htm`,同时将读取的内容通过变量`info`传递过去.

下面我们来编辑这个页面,
`$vim ~/temp/addtest/files/usr/lib/lua/luci/view/addtest_info.htm`
代码如下:

```
<%+header%>
<h2><a id="content" name="content"><%:Addtest Info%></a></h2>
<div id="content_addtest_info">
<textarea readonly="readonly" wrap="off" rows="<%=info:cmatch("\n")+2%>" id="info"><%=info:pcdata()%></textarea>
</div>
<%+footer%>
```

这部分和传统的`html`很类似,我主要是根据其他页面照猫画虎,不是很美观.有机会还要加强这个方面的学习.

---

### Makefile
不知不觉,我们居然已经将代码全部写完了,竟还有点恋恋不舍呢.下面我们用一个`Makefie`文件将它们打包生成一个ipk文件.

```
$vim ~/temp/addtest/Makefile
```

代码如下:

```
include $(TOPDIR)/rules.mk

PKG_NAME:=addtest
PKG_VERSION=1.0
PKG_RELEASE:=1

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)

include $(INCLUDE_DIR)/package.mk

define Package/addtest
	SECTION:=utils
	CATEGORY:=Utilities
	TITLE:=Addtest--print something to /var/addtest
endef

define Package/addtest/description
	It's a test,print something to /var/addtest cyclicaliy
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Package/addtest/postinst
#!/bin/sh
rm -rf /tmp/luci*
endef

define Build/Configure
endef

define Build/Compile
	$(call Build/Compile/Default)
endef

define Package/$(PKG_NAME)/install
	$(CP) ./files/* $(1)/
	$(INSTALL_DIR) $(1)/bin
	$(INSTALL_BIN)  $(PKG_BUILD_DIR)/addtest  $(1)/bin
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
```

Makefile的解释,请参见[拙作](http://www.cnblogs.com/chengyi818/p/4774043.html).我们这里稍作补充.

26~29行

>   由于luci会将模块加载到`/tmp`目录下运行,每次新加载luci模块后,需要执行`$rm -rf /tmp/luci*`.这里表示安装了ipk之后,将会自动执行删除命令,重新载入.

39行

>   $(1)是传入的参数,表示系统镜像目录,你可以将之视为路由器最后的文件系统.所以这句的意思就是将我们`files`下的内容拷贝到路由器的文件系统中.这也是我们为什么要建立一开始那么复杂的目录树的原因.

---

### 编译&安装
简直像裹脚布一样,又臭又长.不要说读了,我自己写的都快有点受不了了.读到这里的人真是辛苦了,下面到了我们收获果实的时候了.
将文件拷贝到源码目录的`package`目录下.其余部分,请参考[拙作](http://www.cnblogs.com/chengyi818/p/4774043.html)

```
$cp ~/temp/addtest ~/openwrt/package
```

把它拷贝到你的开发板中,试试看.

---

### 调试方法
我们当然希望可以一次成功,不过世间不如意之事十之八九.我来谈谈我自己的调试方法.

`src`部分

>   `src`文件下有`Makefile`文件,你可以直接在编译机上执行`$make`生成可执行文件`addtest`,然后在编译机上`src`目录下执行`$./addtest 参数1 参数2`.最后记得执行`$make clean`.

`luci`部分

>   将ipk安装到开发板后,可以通过串口或者ssh的方式登录开发板,然后直接在开发板中修改文件内容,再执行`$rm -rf /tmp/luci*`.最后重新载入设备页面.

---

## 尾记
不知不觉到了分手的时候,竟感觉有些忧桑呢.

### 不足
1. 我自己刚接触学习,难免很多不足
2. 页面输入没有防呆机制

多多包含:)

### 感谢
除了官方文档之外,这两篇博客给我很多指导:
[开发OpenWrt路由器上LuCI的模块](http://www.tuicool.com/articles/zaUNfy),[ openwrt中luci学习笔记](http://blog.chinaunix.net/uid-23780428-id-4367351.html).
我的同事宁财神给我们做了luci的框架介绍,同时在我的调试过程中,给予我很多帮助.
最后感谢管工给出这样一个练习题,虽然很小巧,居然可以贯通整个知识体系.我现在还是为他的高屋建瓴感到惊叹.

### Q&A
在整篇文章学习完成后,我们希望可以回答以下几个问题:

> 1.MVC是什么?各部分有哪些功能?

> 2.怎么在页面上指定位置做出一个子页面.

> 3.怎么将配置写入到路由器中,又怎么读取?

> 4.页面怎么和可执行文件关联起来?或者通俗地说,页面点了一下,开发板怎么就执行了命令.

> 5.ipk怎么生成,安装过程中发生了什么?
