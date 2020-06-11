+ GIC v4.1简介
+ ITS简单演进
+ GIC v4.1新增特性介绍
   + vPE configuration table
   + vPE residency
   + GICR_VPENDBASER
   + VMAPP
   + default doorbell机制
   + vSGI


# GIC v4.1 简介
GIC v4.1主要改变了vLPIs中断的编程接口以及新增支持了vSGIs中断的直接注入。

GIC v4.1和GIC v4.0是完全不兼容的。他修改了GICR的逻辑实现，重写了ITS和GICR的几个寄存器设计，导致这两个版本之间软硬件完全无法通用。在ITS驱动中，很多代码会区分4.1和4.0两条路径。我们可以通过GITS_TYPER.VMAPP这个域对硬件进行简单区分（0:v4.0 1:v4.1）。

# ITS简单演进
在讲解GICv4.1的一些细节前，我们先通过下面这个图，简单梳理下GICv4.1与其他版本的区别以及改进点。
![ITS](https://github.com/Luojiaxing1991/picture/blob/master/ITS_BASE_INTRO.png)

# GIC v4.1新增特性介绍

## vPe configuration table
vPE configuration table可以极大地减少vPE在schedule和deschedule过程中对于GIC的负担，减少切换过程中的刷新同步操作。其中VMAPP的命令执行频率很大幅降低，同时，GICv4.1的硬件设计也让VMOVP的使用频率降低，这个将在vPE resident章节描述。
![vPE configuration table](https://github.com/Luojiaxing1991/picture/blob/master/vPE_configuration_table.png)

## vPE residency
由于vPE configuration table的引入，ITS对于亲和性集群的利用更加得心应手，在集群中的资源共享对于虚拟化场景的vPE迁移有很好的适应性，避免迁移中的资源更新同步的操作。
![vPE residency](https://github.com/Luojiaxing1991/picture/blob/master/vPE_residency.png)

## GICR_VPENDBASER
GICv4.1中，GICR_VPENDBASER虽然沿用了GICv4.0的命名方法，但是由于vLPI Pending table由vPE configuration table进行管理。所以这个寄存器被重新设计。主要用于下面三个功能：
+ 记录当前被scheduel（valid == 1）的vPE的id（vPEID）
+ vPE被deschedule后（valid 1->0）,通过pending last告诉hypervisor这个vPE急需被重新调度
+ vPE被deschedule后（valid 1->0），通过doorbell来指示该vPE的default doorbell是否需要使能。

![pending last and doorbell and dirty](https://github.com/Luojiaxing1991/picture/blob/master/VPENDBASER_GIC_4_1.png)

## VMAPP
VMAPP一般会在vPE被创建后发送给ITS，它的目的有两个：1、将vPEID与某一个GICR进行绑定；2、更新vPE configuration表中，vPEID所对应的entry。

### VMAPP发布给数个ITS时的处理
![VMAPP for several ITS](https://github.com/Luojiaxing1991/picture/blob/master/VMAPP_for_sereral_ITS.png)

## default doorbell机制
在GICv4.1中，doorbell机制分为了两种：individual doorbell(GICv4.0中使用的doorbell机制)和default doorbell(GICv4.1中新增的doorbell机制)

是否支持individual doorbell机制可通过GITS_TYPER.nID进行配置
GITS_TYPER.nID==1 不支持individual doorbell机制
GITS_TYPER.nID==0，支持individual doorbell机制

从协议中可以知道，当 GITS_TYPER.nID==1(no individual doorbell，命令VMAPTI和VMAPI中的 Dbell_pINTID域会被当做1023，也就是无效值

每一个vPE都有一个default doorbell，由VMAPP命令指定doorbell的INTID

### individual doorbell
individual doorbell在GICv4.1中仍然被沿用，由于它是通过VMAPI或者VMAPTI设置的，这就意味着它是每个vIRQ一个doorbell irq的，而且这种doorbell的优先级高。例如，default doorbell可以让vPE被标记为运行，但是无法保证里面被调度，但是可以通过individual doorbell可以指示hypervisor立即调度vPE运行。

### default doorbell
default doorbell和individual doorbell有一些本质的区别是 default doorbell是每个vPE一个，而且它有一些触发的限制，例如：如果vPE已经处于no idle(已经告知hypervisor这个vPE需要被再次schedule)，那么就不会在往vPE上发default doorbell IRQ。同一个default doorbell处于pending状态时，也不会重新发送doorbell IRQ（协议限制，一个vPE在被移除和重新调度之间，default doorbell只能被发一次）。

这就决定了default doorbell的数量会小于individual doorbell，对于hypervisor来说，负担就减轻了，但是对于一些优先级比较高的中断，还是采用individual doorbell比较好。

![default doorbell IRQ](https://github.com/Luojiaxing1991/picture/blob/master/ITS_default_doorbell_IRQ.png)
