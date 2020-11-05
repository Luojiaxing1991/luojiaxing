+ RVIC简介
+ 支持的Hypervisor类型
+ RVID简介

# RVIC简介
Reduced Virtual Interrupt Controller(RVIC)是一个半虚拟化(Para virtualized)的中断控制器，专门提供给VM用于支持基础的中断服务。

根据半虚拟化的概念，GuestOS知道自己运行在虚拟环境中，可调用宿主机上专用的系统调用（Hypercall）来访问宿主机上的物理硬件。以RVIC为例：

![RVIC Para virtualized](https://github.com/Luojiaxing1991/picture/blob/master/RVIC_para_virt.png)

RVIC的目的是为了提供一个轻量级的中断控制器，从而可以作为Trusted Hypervisor的一部分嵌入到Hypervisor中。其他Hypervisor不需要的功能部分可以移动到VM或者untrusted Hypervisor中。

每一个VPE都会有一个RVIC的实例。中断处理，在硬件层面的最后一层都是由PE（vPE）写CPU接口（vCPU接口）的寄存器ICC_（ICV_）来通知CPU(vCPU)对中断进行处理。PE的上游就是中断控制器。但是对于vPE而言，在RVIC尚未问世的阶段，它的上游是Hypervisor或者ITS，如果硬件体系支持RVIC后，那么vPE的上游就变成了RVIC。谁来与vPE对接，这个并不重要，关键是对虚拟中断的管理是否变得更加轻松了。

如果RVIC中存在一个或者多个pending的中断，会通过虚拟的中断异常（virtual IRQ exception）来告知vPE。当vPE处理中断异常时，会向RVIC这里询问是哪个中断触发了异常。每一个RVIC中每一个中断都有一个独立的编号（INTID）来标识。（INTID？ GIC里面是有vINTID的，这个需要明确下）

RVIC支持两种类型的中断：
1. Trusted interrupts
2. Untrusted interrupts

# 支持的Hypervisor类型
RVIC可支持整体型Hypervisor(monolithic hypervisor)和分离型Hypervisor(Split-mode hypervisor)。分离型Hypervisor根据中断的特权级别分为Trusted Hypervisor和Untrusted Hypervisor。对于整体型Hypervisor则不做区分。

# RVID简介
