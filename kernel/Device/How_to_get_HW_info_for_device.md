+ 简介
+ 资源(resource)与属性(property)
+ 基于platform框架的资源获取
+ 寄存器基地址获取

# 简介
当驱动和一个外设绑定后，当做完一些内存申请后，以及设备初始化之前，首先需要获取硬件信息。最基本的硬件信息包括寄存器基地址，硬件的一些特定信息两类。这个文档主要描述如何为一个平台设备获取上述的硬件信息。

# 资源(resource)与属性(property)
从APCI的DSDT表中在描述硬件信息时，非常常用的两个宏分别是_CRS和_DSD。他们所对应的就是设备的资源与属性。

资源一般包括设备寄存器内存空间的描述，包括基地址，长度范围,基础API定义在lib/devres.c。

属性一般指设备特有的数据，例如PGIO管脚个数，基础API定义在drivers/base/property.c。属性会区分APCI和DT，所以在drivers/acpi下，也有一个property.c的文件，里面主要是APCI的属性API。

# 基于platform框架的资源获取
platform框架的API位于drivers/base/platform.c，如果是一个platform设备，通过platfrom框架可以屏蔽掉device resource相关底层描述。通过struct platform_device以及资源的index即可拿到对应的资源信息。例如devm_platform_ioremap_resource(),通过这个函数可以获取该平台设备的第n个设备寄存器地址。ioremap指定了index指定的是寄存器地址资源。

# 寄存器基地址获取
