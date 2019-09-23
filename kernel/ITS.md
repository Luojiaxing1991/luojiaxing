# ITS maintainer's kernel git repo
https://git.kernel.org/pub/scm/linux/kernel/git/maz/arm-platforms.git/  

# ITS MBI transport 
![Image text](https://github.com/Luojiaxing1991/picture/blob/master/GICv3_MBI_transport.png)

# Debug ITS

我们可以在irq-gic-v3-its.c的文件开始加上  
#define DEBUG 1  
这样追加pr_debug这一部分的打印  

e.g
[    8.435707] hisi_sas_v3_hw 0000:74:02.0: Adding to iommu group 0  
[    8.461467] scsi host0: hisi_sas_v3_hw  
[    9.683463] ITS: alloc 9920:32  
[    9.686509] ITT 32 entries, 5 bits  
[    9.690044] ID:0 pID:9920 vID:23  
[    9.693263] ID:1 pID:9921 vID:24  
[    9.696480] ID:2 pID:9922 vID:25  
[    9.699696] ID:3 pID:9923 vID:26  
[    9.702911] ID:4 pID:9924 vID:27  
[    9.706128] ID:5 pID:9925 vID:28  
[    9.709344] ID:6 pID:9926 vID:29  
[    9.712560] ID:7 pID:9927 vID:30  
[    9.715776] ID:8 pID:9928 vID:31  
[    9.718990] ID:9 pID:9929 vID:32  
[    9.722207] ID:10 pID:9930 vID:33  
[    9.725510] ID:11 pID:9931 vID:34  
[    9.728813] ID:12 pID:9932 vID:35  
[    9.732116] ID:13 pID:9933 vID:36  
[    9.735419] ID:14 pID:9934 vID:37  
[    9.738721] ID:15 pID:9935 vID:38  
[    9.742024] ID:16 pID:9936 vID:39   

# LPI 

locality-specific peripheral interrupt: 特定的本地外设中断。
LPI中断是GIC v3新增支持的，跟其他中断在很大程度上有所不同。
尤其，LPI中断一定是MBI（message-based interrupts）,这种中断的配置是保存在内存空间中的表格（tables）中，而不是在寄存器里面。

LPI只有当 GICD_CTLR.ARE_NS == 1才会支持。那就是说只有EL1能执行 LPI的内存访问和配置修改？

# Interrupt Identifiers

每一个中断源都需要通过一个ID来标示它，简写为 INTID，这个ID可以理解为hw-irq，这与我们在系统里面看到的v-irq的值时不同的，这点请知悉。
LPI中断被划分到8192~greater这一域段。

这里后续可以把 GICv3_software_overview_official_release_B.pdf中的 “Table 3 Interrupt ID ranges”放上来

# how LPIs are signaled to the GIC
GICv3 追加了对 MBI中断的支持。我们可以通过写GIC寄存器的方式 设置/清除一个中断。具体来说是写ITS寄存器组，GITS_TRANSLATER。（后续应该把写哪些寄存器，写什么内容补充到这里，这个描述太简单了，到底写一个寄存器还是多个，都不清楚）

后续把 GICv3_software_overview_official_release_B.pdf中的 “Figure 3 Message-based interrupt transported over the interconnect”放上来

# Interrupt state machine

通常中断有 inactive，active，pending，active and pengding 4种状态。

但是LPI只有 inactive, pending两种状态。状态信息保存在内存中。
当LPI被确认后，状态从pending转移到inactive。在LPI pending table中，一个LPI占用一个bit来标示状态。（但是对于同一个LPI，如果中断上报非常频繁，ITS确认不过来的情况下，如何处理，并没有说明，后续可以添加到这里）

# Security model

# Distributor interface

ITS 不涉及 distributor，所以暂不关注

# Redistributor interface

对于ITS，转发器需要知道各个LPI中断的中断属性和pending state的内存基地址。 分发器只负责把IRQ分发到对应的转发器，是否进一步转发到PE，由转发器来决定。

# CPU interfaces
每一个转发器都会连接到一个CPU接口，CPU接口可以提供 如下的可编程接口：
1. 通用配置和使能中断处理的配置
2. 确认一个中断
3. 执行中断的优先级降低和deactivation
4. 给PE设置中断优先级mask
5. Defining the preemption policy for the PE
6. Determining the highest priority pending interrupt for the PE

# Configuring LPIs

## ITS

### Operation of an ITS

一个外设通过写入寄存器 GITS_TRANSLATER 来产生一个LPI。这次写操作主要提供下面两个数据：
1. EventID
   这个ID用来标示外设产生了哪一个中断。这个ID有可能跟INTID一致，也有可能由ITS转换为INTID
2. DeviceID
   这个ID用来标示外设。

IST会把EventID转换为INTID，转换规则由外设决定，这就要求必需配合DeviceID。所以 ITS的INTID = EventID + DeviceID。

LPI中的几个INTIDs，它们会被组进各自的集合中。一个集合中的所有INTID都会路由到一个指定的转发器上。

一个ITS设备会用下面三种表来处理LPI的转换和路由。
1. Device Tables
   这个表用来通过DeviceID找到对应的ITT表
2. Interrupt Translation Tables
   这个表通过Device ID和 EventID来找到对应的 INTID。同时也会标示这个INTID属于哪个集合。
3. Collection Tables
   这是集合与转发器对应的表

【可以把 sofgware_overview 文档的 Figure 17 An ITS forwarding an LPI to a Redistributor 图加到这里】

### The command queue
我们可以通过储存在内存里的命令队列来控制ITS。这个是循环使用的buffer，通过下面三个寄存器来定义：
1. GITS_CBASER
这个寄存器定义了cq的基地址和长度。cq的长度必须是64KB对齐。cq里面的每个entry都是32bytes。

2. GITS_CREADR
指向ITS即将处理的下一个命令的index

3. GITS_CWRITER
这个寄存器指向下一个可写的entry。

### Initial configuration of an ITS

1. 为设备和集合表申请内存空间
寄存器 GITS_BASER[0~7]用来定义ITS设备以及集合表的基地址和大小。软件可以通过这些寄存器去发现ITS支持的表的数量和类型。
我们知道ITS的寄存器位于内存中，且由BIOS申请，并且把物理地址保存在MADT表中。ITS在MADT表中的物理地址，其实就是寄存器的基地址。

# ACPI probe
its_acpi_probe() 是用来对ITS设备进行probe的函数。
BIOS在扫描ITS设备时已经为每个ITS设备都分配了内存空间，这段内存空间的首地址被存储在MADT表中。BIOS还会为ITS设备创建SRAT表，
SRAT表中会存储ITS相关NUMA节点信息。由于ITS没有专有内存，所以对应设备内存应该尽可能保存在相应的node节点下。包括后续ITS设备创建
各类表格也需要考虑内存在哪个NUMA节点。

ITS可以使用ACPI的方式进行初始化。之所以他需要通过ACPI去初始化，是因为ITS在GIC里面，但是它的初始化区别于DT（device tree），所以它只能通过其他方式进行初始化，比如ACPI。
它可以通过MADT表来收集寄存器的基地址。

## MADT
Multiple APIC Description Table的略称，APIC又是Advanced Programmable Interrupt Controller的略称，由于ITS是中断控制器中的设备，所以他的信息是存储在MADT中的。

ITS的MADT表中 主要存储了如下信息：
1. GIC ITS ID：In a system with multiple GIC ITS units, this value must be unique to each one.
2. Physical Base Address： The 64-bit physical address for the Interrupt Translation Service

上述该物理地址（地址连续）需要通过ioremap()转换为驱动可以访问的虚拟地址。

## SRAT
ACPI SRAT table是ACPI Static Resource Affinity Table的缩写，它的作用是保存处理器和内存的拓扑信息。
对于ITS来言，NUMA节点这种与亲和性关联的信息就位于SRAT表中。

## quiescent
ITS的静默状态，该标志位位于GITS_CTLR，用来指示当GITS_CTLR.Enabled == 0时，所有ITS操作已经完成。意味着ITS没有任何业务在运行。


## IORT


