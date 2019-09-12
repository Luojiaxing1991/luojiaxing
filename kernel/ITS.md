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

# ACPI probe

ITS可以使用ACPI的方式进行初始化。之所以他需要通过ACPI去初始化，是因为ITS在GIC里面，但是它的初始化区别于DT（device tree），所以它只能通过其他方式进行初始化，比如ACPI。
它可以通过MADT表来收集寄存器的基地址。

## MADT
Multiple APIC Description Table的略称，APIC又是Advanced Programmable Interrupt Controller的略称，由于ITS是中断控制器中的设备，所以他的信息是存储在MADT中的

## SRAT
ACPI SRAT table是ACPI Static Resource Affinity Table的缩写，它的作用是保存处理器和内存的拓扑信息。

