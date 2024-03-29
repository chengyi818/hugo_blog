---
title: "Arm64异常模型"
date: 2021-11-14T21:50:55+08:00
lastmod: 2021-11-14T21:50:55+08:00
draft: false
description: "Arm64异常等级翻译学习"
tags: ["Arm64", "Exception Level"]
categories: ["Os"]

---

无论在x86还是arm中都有类似CPU执行等级的设计,较高的执行等级可以管理更多的资源,执行更多的指令. 而较低权限的执行等级则受限于硬件机制设计,从而可以在某种安全边界内运行.
本文主要翻译自Armv8-A官方文档,用于学习Arm的执行等级设计. 同时类比x86系统,相互印证.

<!--more-->

# x86 i386的执行等级设计
我们简单举例说明x86的执行等级设计, 在操作系统中,内存是非常关键的一类资源. 操作系统希望用户态app拥有自己的内存,但是又不能去访问不属于自己的内存.那么x86的CPU是如何支持这一目的的呢?

首先,i386通过CPL寄存器,标记了两种CPU执行等级, 0表示内核态, 3表示用户态. MMU读取的页表物理地址保存在CR3寄存器,该寄存器只有内核态允许写. 页表不仅包含了用户态app虚拟地址和物理地址的对应关系,同时也包含了该对应关系的访问权限. 操作系统在内核态会为用户态app设置好其对应的页表,然后将CPU控制权转让给app.

在app接过cpu控制权,开始执行其指令时,如果越过了页表中设置的映射要求,那么MMU会发出中断,通过操作系统事先注册好的中断处理程序来处理.

类似地,Arm也有自己的权限等级设计. 不仅如此,Arm为了支持虚拟化和TEE,其执行等级更多一些.

---

# Arm64 异常模型
## 特权等级和异常等级
因为Armv8-A的特权等级只有在异常发生或者返回时发生改变,因此在Armv8-A架构中特权等级和异常等级是同一个概念.每个异常等级都有自己的编号,权限越高,数字越高.
![privilege_levels](https://documentation-service.arm.com/static/614c7d7bd5c3af0155495144?token=)

如上图所示,通常用户态应用程序运行在EL0,操作系统运行在EL1, 虚拟机hypervisor运行在EL2, EL3一般运行底层固件.CPU架构本身没有强制绑定这样的软件模型,但是一般情况下都是如此.

### 异常等级分类
有两类资源的访问受到特权等级的限制,一是内存,二是CPU的寄存器.

### 异常等级: 内存
Armv8-A支持虚拟内存系统,其中内存管理单元(MMU)允许软件为内存区域配置属性.这些属性包括读/写权限,可以为特权和非特权访问单独配置访问权限.当处理器在EL0执行等级时内存访问将根据非特权访问权限进行检查.当执行等级为EL1/EL2/EL3时,内存访问将根据特权访问权限进行检查.

由于内存访问权限是由软件通过MMU使用的页表进行配置的,因此也应该考虑对这些页表进行配置所需的特权. MMU配置存储在系统寄存器中,访问这些寄存器的权限同样也受当前异常级别控制.

### 异常等级: 寄存器
Arm CPU的配置保存在一系列系统寄存器上,这些系统寄存器共同定义了当前CPU的上下文状态.很明显,访问这些系统寄存器必定受到当前异常等级的限制.

系统寄存器的名称反映了访问该寄存器所需的最低权限. 比如`TTBR0_EL1`寄存器保存了EL0和EL1使用的页表基地址.该寄存器的最低访问等级为EL1,如果EL0尝试访问该寄存器,则MMU会抛出异常.

Arm体系结构有许多具有概念上相似功能相近的寄存器,它们的名称仅在异常级别后缀上有所不同.这些是独立的不同的寄存器,在指令集中有自己的编码,并将在硬件中单独实现. 例如: 以下寄存器为不同的异常等级内存转换的MMU配置.寄存器具有相似的名称以反映它们执行相似的任务,但它们是完全独立的寄存器,具有自己的访问原语.
* SCTLR_EL1 - Top level system control for EL0 and EL1
* SCTLR_EL2 - Top level system control for EL2
* SCTLR_EL3 - Top level system control for EL3

备注: 因为EL0作为用户态程序根本没有必要配置系统寄存器,所有可以看到EL0和EL1共用MMU配置寄存器. 对于其他的系统寄存器也同样如此.

较高的异常级别有权访问较低级别的寄存器.例如:EL2有权访问`SCTLR_EL1`.在操作系统中,一般只会修改同样异常级别的配置.然而,更高的特权级别有时会访问与较低异常级别的寄存器,例如:实现虚拟化功能或在上下文切换或电源管理操作.

----

## 执行状态和安全状态
Armv8-A处理器的执行状态由异常级别和其他两个重要状态决定.当前执行状态定义了通用寄存器的标准宽度和可用指令集.执行状态也会影响内存模型的各个方面以及如何管理异常.

当前安全状态控制当前有效的异常级别,当前可以访问的内存区域以及这些访问在系统内存总线上的表示方式.下图显示了不同异常级别和安全状态下的执行状态:
![execution_state](https://documentation-service.arm.com/static/614c7d7bd5c3af0155495145?token=)

### 执行状态
Armv8-A有两种执行状态:
* AArch32: 32位执行状态,兼容Armv7-A. 使用Arm32和Thumb32指令集,寄存器32位宽.
* AArch64: 64位执行状态,使用AArch64指令集.寄存器64位宽.

### 安全状态
Armv8-A支持两种安全状态,以进一步划分软件.
* 安全状态: Processing Element(PE)可以访问安全和非安全物理地址空间.PE可以访问安全和非安全系统寄存器. 软件只能捕获安全中断
* 非安全状态: PE只能访问非安全物理地址空间.PE只能访问非安全系统寄存器.软件只能捕获非安全中断.

安全状态主要是为了Trustzone服务的,具体细节可以参考[TrustZone for Armv8-A](https://developer.arm.com/documentation/102418/latest/)

### 改变执行状态
PE只能在CPU复位或者异常等级改变时,改变执行状态.

复位时的执行状态由`IMPLEMENTATION DEFINED`机制确定.一些CPU实现改变了重置时CPU的执行状态.例如,Cortex-A32将始终重置为AArch32状态.在Armv8-A的大多数实现中,复位后的执行状态由在复位时采集到的信号控制.这允许在SOC级别控制复位执行状态.

当PE在Exception级别之间发生变化时,也可以更改执行状态. AArch32和AArch64之间的转换必须遵守如下规则.
* 当从较低的异常级别移动到较高级别时,执行状态可以保持不变或更改为AArch64.
* 当从较高的异常级别移动到较低级别时,执行状态可以保持不变或更改为AArch32.

结合这两条规则,我们可以发现在64位层上可以运行32位层.但是反过来就不可以.如下图举例所示:
![changing_exec_status](https://documentation-service.arm.com/static/614c7d7bd5c3af0155495147?token=)

上图仅仅是OS和APP的示例,对于其他层同样如此.比如32bit的EL2 hypervisor只能运行32bit的EL1 VM.

### 改变安全状态
EL3总是被认为是在安全状态下执行的.使用`SCR_EL3`(System control register),EL3代码可以更改所有较低异常级别的安全状态.如果软件使用SCR_EL3更改较低异常级别的安全状态,则PE将不会更改安全状态,直到它更改为较低的异常级别.

### 异常等级和执行状态的实现
Armv8-A架构允许SOC实现选择是否实现所有异常级别,并为每个实现的异常级别选择允许哪些执行状态.

EL0和EL1是唯一必须实现的异常级别. EL2和EL3是可选的. 选择不实施 EL3 或 EL2 具有重要影响.

EL3 是唯一可以更改安全状态的级别. 如果实现选择不实现 EL3,则该 PE 将无法访问安全状态.
同样,EL2 包含许多虚拟化功能.没有EL2的实现则不可以访问这些功能.该架构的所有当前 Arm 实现都实现了所有异常级别,如果没有所有异常级别,就不可能使用大多数标准软件.

SOC实现还可以选择异常级别允许的执行状态.如果某个异常级别允许AArch32,则所有较低的异常级别都必须允许AArch32.例如，如果 EL3 允许 AArch32，那么它必须在所有较低的异常级别上都被允许。

许多SOC实现允许所有执行状态和所有异常级别,但也有限制的SOC实现.例如,Cortex-A32只在任何异常级别允许AArch32.一些更新的SOC实现,例如Cortex-A55,实现了所有异常级别,但只允许AArch32在EL0.其他异常级别EL1/EL2/EL3必须是AArch64.

---

## 异常类型
异常是可以导致当前正在执行的程序被挂起,同时导致状态发生变化并执行特定代码来处理该异常的任何事件.其他处理器架构可能将此描述为中断.在 Armv8-A 架构中,中断特指外部产生的异常. Armv8-A架构将异常分为两大类:同步异常和异步异常.

### 同步异常
同步异常是可能由刚刚执行的指令引起或与之相关的异常.这意味着同步异常与执行流同步.

尝试执行无效指令可能会导致同步异常,无论是当前异常级别不允许的指令还是已禁用的指令.由于地址未对齐或MMU权限检查失败,内存访问也可能导致同步异常.由于这些错误是同步的,因此可能在尝试访问内存之前发生异常.内存访问也可能产生异步异常,*内存管理指南*(Memory Management guide)中更详细地讨论了内存访问错误.


Armv8-A 架构有一系列异常生成指令:SVC(supervisor call instruction)/HVC(Hypervisor call instruction)和SMC(secure monitor call instruction).这些指令不同于普通的无效指令,因为它们用于切换到不同的异常级别.这些指令用于实现系统调用接口,以允许较低特权的代码从较高特权的代码请求服务.

Debug异常也属于同步异常.Debug异常在*Debug概览*(Debug overview guide)中讨论.

### 异步异常
某些类型的异常是在外部生成的,因此与当前指令流不同步.这意味着无法预测何时会发生异步异常. Armv8-A架构只要求它在有限的时间内发生.异步异常也可以暂时屏蔽.这意味着在发生异常之前,异步异常可以处于挂起状态.

异步异常包括:
* 物理中断: SError(System Error), IRQ(interrupt request), FIQ(fast interrupt request)
* 虚拟中断: vSError(Vitrual System Error), vIRQ, vFIQ

物理中断是由于PE外产生的信号而产生的.虚拟中断可以由外部产生或由在EL2执行的软件产生.虚拟中断将在*虚拟化指南*(Virtualization guide)中讨论.

### 物理中断: IRQ和FIQ
Armv8-A 架构有两种异常类型,IRQ 和 FIQ,用于生成外设中断. 在 Arm 架构的其他版本中,FIQ 用作更高优先级的快速中断. Armv8-A不同,其中FIQ与IRQ具有相同的优先级.

IRQ和FIQ具有独立的路由控制,通常用于实现安全和非安全中断.如通用中断控制器指南(Generic Interrupt Controller guide)中所述.

### 不可屏蔽中断(NMI)
2021版扩展Armv8.8-A和 Armv9.3-A 添加了不可屏蔽中断 (NMI) 支持.当支持和启用时,中断可以作为具有超优先级传递给处理器.具有超优先级的中断被归类为NMI,即使 PSTATE 异常掩码也无法阻止它被传递.

NMI 有一些限制.所有中断,包括 NMI,在第一次发生中断异常时都会被屏蔽.这是为了允许软件在被后续中断覆盖之前保存所需的状态.在某些时候,软件无法处理任何中断,包括 NMI.为了解决这个问题,添加了一个新的 PSTATE 掩码 ALLINT.这允许软件在屏蔽无中断,大多数中断（但不包括 NMI）和所有中断（包括 NMI）之间进行选择.

支持三种 NMI 支持模型:
1. SCTLR_ELx.NMI = 0: 禁用 NMI 支持.可以使用 PSTATE.I 和 PSTATE.F 屏蔽中断.
2. SCTLR_ELx.NMI = 1: 启用 NMI 支持.
> 1. SCTLR_ELx.SPINTMASK = 0: NMI 被 PSTATE.ALLINT 掩码屏蔽.
> 2. SCTLR_ELx.SPINTMASK = 1: NMI 被 PSTATE.ALLINT 掩码或当目标异常级别使用 SP_ELx 时被屏蔽.

NMI支持要求系统的中断控制器能够呈现具有超优先级的中断.如果使用 Arm 的通用中断控制器,则 GICv3.3 和 GICv4.2 中添加了对 NMI 的支持.

### SError
SError 是一种异常类型,由内存系统生成以反应错误的内存访问. SError的典型用途如: 以前称之为外部异步中止.例如已通过所有MMU检查但在内存总线上遇到错误的内存访问. 这可能是异步报告的,因为该指令可能已经执行完成. SError中断也可能由某些RAM上的奇偶校验或纠错码(ECC)检查引起,例如内置cache.

---

## 处理异常
当发生异常时,当前程序执行流被打断.程序单元(PE)将更新当前状态并跳转到向量表中的某个位置.通常这个位置将包含异常通用处理代码,用于将当前程序的状态压入栈中,然后进一步跳转到具体处理代码.如下图所示:
![exception_handle](https://documentation-service.arm.com/static/614c7d7bd5c3af0155495149?token=)

### 中断术语
异常发生时处理器所处的状态称为异常发生起始状态.异常发生后PE立即所处的状态是异常发生目的状态.例如,异常状态可能是从 AArch32 EL0 到 AArch64 EL1.

Armv8-A架构具有从异常返回的指令.在那种情况下,执行该指令时PE所处的状态就是异常返回起始状态.异常返回指令执行后的状态就是异常返回的目的状态.

每一种异常类型都将路由到特定的异常等级. 异步异常可以被路由到不同的异常等级.
Ps: 这句没看懂, Each exception type targets an Exception level.

### 发生异常
当发生异常时,当前执行上下文必须被保存,以便在异常处理完成后返回. PE会自动保存异常返回地址和当前执行上下文`PSTATE`.

当前上下文使用的通用寄存器由软件自行负责保存.然后PE将会根据异常类型更新`PSTATE`的内容,然后跳转到中断向量表中的异常处理代码.

跳转起始时的上下文`PSTATE`将会保存在`SPSR_ELx`寄存器,同时异常返回地址保存在`ELR_ELx`,其中x表示异常跳转的目的等级.
比如从EL0跳转到EL1,那么EL0的上下文保存在`SPSR_EL1`.返回到EL0的执行地址保存在`ELR_EL1`.

### 异步异常路由
三种物理中断类型可以独立路由到任何一个异常级别,EL1/EL2或EL3.下图以 IRQ 为例:
![interrupt_routing](https://documentation-service.arm.com/static/614c7d7bd5c3af0155495143?token=)

路由是通过`SCR_EL3`和`HCR_EL2`寄存器控制的.其中`SCR_EL3`的配置会覆盖`HCR_EL2`.这些路由允许中断类型被路由到不同的处理程序.

路由到比正在执行的级别更低的异常级别的异常被隐式屏蔽.异常将被挂起,直到PE更改为等于或低于路由到的异常级别.

### 异常路由配置
处理异常的执行状态由更高的异常级别决定.假设实现了所有异常级别,下表显示了如何确定执行状态.
| 异常目的状态 | 控制寄存器         |
| :--          | :--                |
| 非安全EL1    | HCR_EL2.RW         |
| 安全EL1      | SCR_EL3 或 HCR_EL2 |
| EL2          | SCR_EL3.RW         |
| EL3          | EL3重置            |

### 从异常处理返回
软件可以通过执行AArch64的`ERET`指令来从异常返回.这将导致根据`SPSR_ELx`的值来配置返回的异常级别,其中`x`是异常返回的起始级别. `SPSR_ELx`包含要返回的目标级别和目标执行状态.

注意 SPSR_ELx 中指定的执行状态必须与 SCR_EL3.RW 或 HCR_EL2.RW 中的配置匹配,否则会产生非法异常返回.

执行 ERET 指令时,执行状态将从 SPSR_ELx 恢复,程序计数器将更新为 ELR_ELx 中的值.这两个更新是CPU自动且不可分割地执行,这样PE就不会处于未定义状态.

### 异常执行栈
在 AArch64 中执行时,该架构允许选择两个堆栈指针寄存器; SP_EL0 或 SP_ELx,其中 x是当前异常级别.例如,在 EL1 中可以选择 SP_EL0 或 SP_EL1.

在一般执行期间,所有代码都使用 SP_EL0.当发生异常时,则选择 SP_ELx.这样为异常处理维护单独的函数栈.这对于在处理由栈溢出引起的异常时维护有效堆栈很有用.


---

## 中断向量表
在Armv8-A中,向量表是包含指令的普通内存区域. 处理元素(PE)将向量表的基地址保存在系统寄存器中,并且每个异常类型都有一个特定的相对于该基址的偏移量.

每个特权异常级别都有自己的向量表,由向量基地址寄存器VBAR_ELx定义,其中 x 是 1/2/3.VBAR 寄存器的值在复位后未定义,因此必须在启用中断之前对其进行配置.

向量表格式如下:
![vector_table](https://documentation-service.arm.com/static/614c7d7bd5c3af0155495148?token=)

每种异常类型都可以路到到四个位置之一,具体取决于异常级别的状态.

---

# Ref
1. Arm官方文档[Learn the architecture: AArch64 Exception model](https://developer.arm.com/documentation/102412/0100)
2. Memory Management guide
3. Debug overview guide
4. Virtualization guide
5. Generic Interrupt Controller guide
