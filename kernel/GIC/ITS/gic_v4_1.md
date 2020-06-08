+ GIC v4.1简介
+ GIC v4.1新增特性介绍
   + vPE configuration table
   + GICR_VPENDBASER
   + default doorbell机制
   + vSGI
   + VMAPP
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

## GICR_VPENDBASER
GICv4.1中，这个寄存器其实被完全重写了

这个寄存器被用于描述当前被scheduel的vPE信息以及刚刚被deschedule的上一任vPE的残留信息（pending last，）

![pending last and doorbell and dirty](https://github.com/Luojiaxing1991/picture/blob/master/VPENDBASER_GIC_4_1.png)
