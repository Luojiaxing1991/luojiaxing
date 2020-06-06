# ITS简介

# ITS基本流程

下图描述了ITS在GICv3，GICv4.0和GICv4.1之间的处理流程。ITS需要感知每一个LPI中断，并将LPI中断及时通知到对应的处理器（cpu/vcpu）。在虚拟化的场景中，vcpu会在cpu之间进行调度，ITS也需要准确跟踪vcpu当前的宿主，确保LPI正确的发送到对应的cpu上进行排队。

![ITS base function](https://github.com/Luojiaxing1991/picture/blob/master/ITS_BASE_INTRO.png)

