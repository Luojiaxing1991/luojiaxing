# GIC基本架构
GIC的组件主要包括下述5类：
- GICD(GIC Distributor)
  按照协议说明，GICD主要用于处理SPI和PPI中断，兼任SGI中断的中转路由
- GICR(GIC Redistributor)
  按照协议说明，GICR位于GICD和CPUIF之间，用于转发SPI，PPI，SGI，并且接受LPI中断
- CPUIF(CPU interface)
  对接GICC，完成中断上报
- ITS
  产生LPI中断，并路由到对应的GICR
- GICC
  汇聚中断，通过IRQ/FIQ上报给CPU
  
Interrupt routing infrastructure(中断路由基础架构)

![Image text](https://github.com/Luojiaxing1991/picture/blob/master/IRI_GIC.png)

由上图可知，GICD的路由（如果支持路由）以x.y.z.i这样的路由表进行的，所以你看到CPU是以cluster为集合的。

index | meaning
:---- | :-----
x     | 预留
y     | SCCL index
z     | cluster index
i     | CPU index

## SCCL
super code cluster的简称，与NUMA节点属于同一粒度。一个SCCL内部，L3 cache是互通的，所以数据交互速度很快。SCCL之间的L3 cache可以可以交互数据，速度比DDR读写块，但是如果跨片访问，需要通过IO DIE，走总线，因此速度最慢，慢于DDR读写。

SCCL与SICL的组合（个数组合），可以提供多种芯片设计方案。

![Image text](https://github.com/Luojiaxing1991/picture/blob/master/SCCL%26CCL.png)

## 组件的互通关系
GICD、GICR、ITS都被包括在IRI里面，用于路由，既然是路由，那么需要先明确这几个组件之间的互通关系
- Y: 硬件线连接
- N: 不连通
- B: 总线连接

From/To | GICD | GICR | ITS | CPUIF
:-----: | :-:  | :-:  | :-: | :-:
GICD    | B    | Y    | N   | N
GICR    | N    | B    | N   | Y
ITS     | N    | B    | N   | N
CPUIF   | B    | N    | N   | N

简单归纳  ->(hardware wire)  --> bus
- SPI： Device -> GICD -> GICR -> CPUIF
- PPI:  Device -> GICR -> CPUIF
- SGI:  CPU -> CPUIF --> GICD -> GICR -> CPUIF
- LPI:  Device --> ITS --> GICR -> CPUIF

# 通过ITS上报中断流程
笔者主要研究ITS，所以目前先把ITS作为重点来讲。
