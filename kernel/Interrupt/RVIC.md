+ RVIC简介
  + 支持的Hypervisor类型
  + signaling
  + RVIC enable
  + Trusted and Untrusted IRQ
+ RVID简介


# RVIC简介
Reduced Virtual Interrupt Controller(RVIC)是一个半虚拟化(Para virtualized)的中断控制器，专门提供给VM用于支持基础的中断服务。

根据半虚拟化的概念，GuestOS知道自己运行在虚拟环境中，可调用宿主机上专用的系统调用（Hypercall）来访问宿主机上的物理硬件。以RVIC为例：

![RVIC Para virtualized](https://github.com/Luojiaxing1991/picture/blob/master/RVIC_para_virt.png)

RVIC支持整体型Hypervisor（monolithic hypervisor）和分离型Hypervisor（split-mode hypervisor）。分离型Hypervisor的实现跨越了数个特权等级（不同特权等级（ELx）在同一或者不同安全状态），从而导致分离型Hypervisor被切分为Trusted Hypervisor和Untrusted Hypervisor。

RVIC的目的是为了提供一个轻量级的中断控制器，从而可以作为Trusted Hypervisor的一部分嵌入到Hypervisor中。其他Trusted Hypervisor不需要的功能部分可以移动到VM或者untrusted Hypervisor中。

每一个VPE都会有一个RVIC的实例。中断处理，在硬件层面的最后一层都是由PE（vPE）写CPU接口（vCPU接口）的寄存器ICC_（ICV_）来通知CPU(vCPU)对中断进行处理。PE的上游就是中断控制器。但是对于vPE而言，在RVIC尚未问世的阶段，它的上游是Hypervisor或者ITS，如果硬件体系支持RVIC后，那么vPE的上游就变成了RVIC。谁来与vPE对接，这个并不重要，关键是对虚拟中断的管理是否变得更加轻松了。

如果RVIC中存在一个或者多个pending的中断，且RVIC的实例当前是使能状态，会通过虚拟的中断异常（virtual IRQ exception）来告知vPE(通过HCR_EL2.vi来产生virtual IRQ exception)。由于导入RVIC，Hypervisor并不直接参与IRQ的处理，因此，无论vPE有没有运行时，当有虚拟中断过来，RVIC都会通过这个虚拟中断异常通知hypervisor尽快调度目标vPE上线处理。当vPE处理虚拟中断异常时，会向对应的RVIC实例询问是哪个中断触发了异常。每一个RVIC中每一个中断都有一个独立的编号（INTID）来标识，这个INTIT是VM范围内独一无二的。（INTID？ GIC里面是有vINTID的，这个需要明确下）

RVIC支持两种类型的中断：
1. Trusted interrupts
2. Untrusted interrupts

每一个RVIC实例都有他们独立的状态，彼此并不共享状态（这与GIC架构中GICR的设计有区别）。这可以降低RVIC实现的复杂性，并且允许VM的中断个数随vPE的个数自然增长。因此RVIC并不支持中断路由。因此，Untrusted IRQ需要通过RVID，而Trusted IRQ需要经由Trusted Hypervisor进行转发。

# 支持的Hypervisor类型
RVIC可支持整体型Hypervisor(monolithic hypervisor)和分离型Hypervisor(Split-mode hypervisor)。分离型Hypervisor根据中断的特权级别分为Trusted Hypervisor和Untrusted Hypervisor。对于整体型Hypervisor则不做区分。

## RVIC enable
RVIC的使能并没有详细介绍，以至于目前不清楚RVIC的实例何时使能。目前估计是，VM里面的vcpu被创建的时候就会使能对应的RVIC实例。

## Trusted and Untrusted IRQ
Trusted IRQ是有Trusted Hypervisor创建，分为两类：
1. 从vPE发给自己或者发给同一个VM下的另外vPE的SGI中断
2. 由Trusted Hypervisor管理的外设可以产生一个Trusted IRQ。例如： per_VPE Arm Generic Timer instance 产生的时间信号中断。

Untrusted IRQ是由Untrusted Hypervisor产生，例如，一个虚拟的外设（网络设备）产生的一个Untrusted IRQ。

RVIC通过分开的INTID域来区分Trusted IRQ和Untrusted IRQ.

RVIC不保证在无限制的条件下实时传递中断。

RVIC不处理VPE的调度。

## signaling


# RVID简介
