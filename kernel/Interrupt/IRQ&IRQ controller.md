+ IRQ
   + irq_chip与irq_common_data与irq_data
   + irq_chip
      + irq_chip的ops   
+ 中断控制器简述
+ Arm架构内的中断控制器
+ 中断控制器的结构体们
   + 结构体们
   + 中断域
      + 中断域与fwnode_handle
      + 线性映射 & 树映射 & 不映射
      + 创建映射与查找映射

# IRQ

## irq_chip与irq_common_data与irq_data
irq_chip和irq_common_data，irq_data的定义都位于irq.h中(include/linux/irq.h)，用作最底层的中断属性描述。irq_data与irq_chip，irq_common_data之间都存在单向的连接。而irq_chip和irq_common_data之间不存在联系。因此中断的重心在于irq_desc->irq_data->irq_chip的单向链条，irq_common_data可认为是irq_data的一些补充说明。

这里应该补充一个图，来说明中断的核心结构体。

irq_data是irq_desc的成员变量。每一个中断都有一个irq_data与之对应。所以它描述的是每一个中断所独有的一些属性，包括中断号，中断寄存器的mask，中断所属的irq_domain，中断所属的irq_chip，中断对应的私有变量chip_data。irq_common_data可以为irq_data提供关于亲和性，NUMA节点，中断标记位等补充信息。共同构成对中断的描述。
``` C
struct irq_data {
   u32 mask;
   unsigned int irq;
   ...
   struct irq_common_data *common;
   struct irq_chip *chip;
   struct irq_domain *domain;
   ...
   void *chip_data;
}
```

irq_common_data是irq_desc的成员变量。它主要为irq_data提供一些补充信息，如下：
``` C
struct irq_common_data {
   unisgned int __private state_use_accessors;
   unsigned int node;
   ...
   struct msi_desc *msi_desc;
   cpumask_var_t affinity;
}
```

irq_chip是中断类的方法集合。主要描述中断的一些通用方法，直接对接中断控制器硬件。详细可以查看后续的irq_chip章节。

# irq_chip

## irq_chip的ops

### irq_ack
调用栈如下：
``` C
desc->handle_irq(desc) - +
                         | -  handle_edge_irq(for example) - +
                                                              | - desc->irq_data.chip->irq_ack()
```
irq_ack主要是通知硬件该中断已被CPU处理中，可以解除该中断的pending状态，并接受新的中断。对于边沿中断，由于边沿触发是短暂的，所以当前触发信号的结束很容易判断。但是对于电平中断，电平信号可能长时间维持高电平，即使软件发出了irq_ack，但是pending状态会持续到中断线解除高电平（或低电平）。

### irq_mask/irq_unmask
设置中断屏蔽的ops，设置中断屏蔽后该中断源无法触发中断处理。通常情况下，当驱动ack了一个中断号，进入中断处理后，需要暂时屏蔽该中断，防止同一个中断之间的嵌套。

### irq_enable/irq_disable

### irq_set_type

# 中断控制器简述

中断是计算机体系中非常重要的一种信号机制。在中断体系中，最简单的功能是硬件打断软件，让它紧急去处理一件事情，而其中负责信号中转的就是中断控制器。

![IRQ controller base function](https://github.com/Luojiaxing1991/picture/blob/master/IRQ_Controller_intro.png)

Arm体系内的中断控制器除了GIC，另外还包括一些级联中断控制器，典型的就是GPIO。

中断的典型流程中，中断源（通常可以是指外设，cpu，软件，时钟等）会通过一些指定的信号（上升沿，下降沿，电平，总线消息）给中断控制器输入一个物理刺激（也包括一些基于信号的中断，如LPI中断），中断控制器确认中断信号后，首先会查看这个中断的基本状态，是否被mask，有没有相同的中断正在排队（pending），应该通知哪个处理器进行处理；然后给这个信号贴上标签（linux IRQ number），放入对应处理器的队列中排队等待（GIC中，GICC是CPU的管家，他负责提醒CPU过来处理中断，并告诉CPU应该处理哪个中断）；当一个中断被CPU确认（acknowledge）后，中断控制器又需要改变这类中断的状态（pending->active/unactive）。

# Arm架构内的中断控制器
ARM架构内的中断控制器有GIC，以及GPIO。

GIC是ARM架构内的通用中断控制器，直接对接CPU，处理的中断可以简单分为线中断和消息中断。GIC比较特殊的地方在于他在绑定硬中断和软中断的时候会遵循firmware的一些配置（例如，mapping软硬中断的时候会用irq_create_fwspec_mapping（））。原因在于服务器设计中，很多线中断会直接连接到GIC，而硬件中断号和软件中断号通过中断向量表已经固定了，并保存在BIOS里面。对应的硬件模块的驱动通过ACPI表可以直接获取到自己的软件中断号，并注册回调函数。对于消息中断这一类，则由PCI模块进行硬中断（这时候是PCI function的event ID）和软中断的绑定。

ARM为GPIO这一类中断控制器提供了一个irq_chip_generic的抽象概念。GPIO的底软驱动可以通过irq_alloc_domain_generic_chips这类通用的API为一个irq_domain创建irq_chip_generic。这个可以为驱动编写减轻负担，不需要考虑中断控制器内部逻辑实现。但是irq_chip_generic有一个限制是只能支持32个中断，因此如果irq_domain中的中断个数比较多，要么为一个irq_domain创建几个irq_chip_generic，要么是自行创建irq_chip，并进行管理，不使用irq_chip_generic。

# 中断控制器的结构体们


## 结构体们
![irq_domain 与 irq_data 与 irq_chip](https://github.com/Luojiaxing1991/picture/blob/master/irq_domain%26irq_data%26irq_chip.png)

irq_domain是一种中断域，用来描述中断的集合；irq_data（ird_desc）主要描述中断对象；irq_chip描述某一类中断的方法。

## 中断域
每一个中断控制器的代码都会首先涉及struct irq_domain这个结构体，以及为自己申请一个irq_domain的资源描述。其他硬件描述如irq_chip_generic和irq_chip都是基于irq_domain来承载。irq_domain翻译过来就是
中断域，从字面理解，它描述了中断控制器的地盘（在linux IRQ number中占据多少个席位）。这个地盘的大小完全由这种中断控制器所拥有的硬中断数量来决定的，简单粗暴，相当于身上有多少伤口，能同时承受多少
电击枪的暴击（抖M值？），它就能拥有多大的地盘。

至于为什么irq_domain会在中断控制器的硬件抽象中占据这么重要的地位呢？笔者认为，对于linux各模块而言，他们眼中能看到的中断只包括三个东西，Linux IRQ number，回调函数，以及对应硬件事件含义（对于自家硬件，肯定知根知底）。
很经济实在，就像女人眼中，结婚就等于房车一样直接。那么，既然客户眼里只有这几个东西，那中断信号是通过电击枪还是通过阿姆斯特朗回旋加速喷气式阿姆斯特朗炮给你一记刺激，对客户他们来说，并不重要，
因此，通过一个mapping关系告诉客户，这个数字，表示之前不久的一个moment，发生了一件什么事情，就是最重要的事情。这就决定了irq_domain的江湖地位，不是老大，也是教主。

### 中断域与fwnode_handle
fwnode_handle作为描述设备集合的原子结构体，它的指针会保存在acpi_device和device_node结构体中，因此，irq_domain与fwnode_handle绑定，可以避免在irq_domain中区分acpi设备和设备树设备。
![irq_domain 与 fwnode_handle](https://github.com/Luojiaxing1991/picture/blob/master/irq_domain%26fwnode_handle.png)

### 线性映射 & 树映射 & 不映射
中断控制器的驱动代码会通过irq_domain_add_*()这样的一个函数来创建和注册中断域，*号其实对应这个三类方法，linear，tree和nomap。（举个栗子，组合起来就是irq_domain_add_linear（））。这三种方式的区别在于两点：首先，nomap跟
linear/tree就不是一路人，它的硬件中断号和软件中断号是相同的，不需要映射，比如powerPC的MPIC架构。其次，linear和tree的区别在于后宫（硬中断）数量级不同，如果你的后宫妃嫔少于256个，那么linear方式就很适合你，一宫之内，为你独尊，
下面的妃嫔都用编号进行编队（hash表），来回也就百十来人，你翻牌子也容易，一张桌子放得下。而如果你的后宫数量级很大，佳丽三千，那翻牌子就不容易了，得设立一些层级结构，东宫西宫，皇后贵妃才人，不然你的桌子根本摆不下
（大数量级下，数的使用可以避免根据最大的硬件中断号来分配一个很大的表）。

PS：对应的三个API函数为： irq_domain_add_linear() irq_domain_add_tree() irq_domain_add_nomap()

### 创建映射与查找映射
创建映射： irq_create_mapping()
查找映射： irq_find_mapping()


