+ GIC v4.1简介
+ GIC v4.1新增特性介绍
   + vPE configuration table
   + GICR_VPENDBASER
   + VMAPP
   + default doorbell机制
   + vSGI
   + 其他变更
+ vPE table详解
   + GICR 三类table介绍
   + vPE table机制
+ default doorbell详解
+ vSGI详解
+ VMAPP详解

# GIC v4.1 简介

# GIC v4.1新增特性介绍

## vPe configuration table
![vPE configuration table](https://github.com/Luojiaxing1991/picture/blob/master/vPE_configuration_table.png)

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


