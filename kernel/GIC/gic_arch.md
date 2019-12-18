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

由上图可知，GICD的路由（如果支持路由）以x.y.z.i这样的路由表进行的。

index | meaning
:---- | :-----
x     | 预留
y     | SCCL index
z     | cluster index
i     | CPU index
