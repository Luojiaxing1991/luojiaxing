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

