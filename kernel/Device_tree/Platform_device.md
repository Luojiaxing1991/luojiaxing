+ 平台设备简介
   + linux内核对于platform的描述
+ struct platform_device
+ module_platform_driver
   + struct platform_driver
   + platform_data
+ 平台设备与平台驱动的绑定
+ ACPI注册平台设备

# 平台设备简介
linux设备驱动模型主要基于总线，设备，驱动三元素实现。linux设备必定会挂在在总线上（PCI，USB，IIC，SPI）等，而每一个设备都有对应的驱动代码。但是，对于集成在SoC里面的外设，一般通过系统内部总线来进行数据交互，而且内部总线通常基于具体产品实现决定，对于linux内核而言，无法进行统一管理。因此，linux内核创建了虚拟总线（platform bus）来对这一类设备进行管理。

## linux内核对于platform的描述
通过查阅 Documentation/driver-api/driver-model/platfrom.rst 可以了解linux内核对平台的定义。如下：

“平台总线是一种伪总线，这种伪总线用于以最小的基础结构连接总线上的设备，例如SoC上的集成外围设备或者某些旧式的PC互连设备，不包括PCI或者USB这样的大型正式指定产品”

“平台设备通常在系统中显式为自治实体的设备。这包括基于传统端口的设备和到外围总线的主机桥，以及大多数集成到SoC的控制器。它们有一个共同的特点是从CPU总线直接寻址。”

“平台驱动遵循标准驱动程序模型约定，其中发现/枚举在驱动程序之外处理，提供probe/remove方法，使用标准确定支持电源管理和关机通知。”

# struct platform_device

# module_platform_driver
这个macro主要用于一些不需要考虑module_init/module_exit的驱动，作为module_init/module_exit的替代效果。

它的入参是 struct platform_driver。

## struct platform_driver
这个结构体作为平台设备驱动的公共框架

## platform_data
平台设备在struct device中有一个platform_data的域段，device框架认为这个是平台设备专用的域，因此不会进行处理，而该域主要由平台设备自行维护。如果platform_data在创建平台设备时候已经分配了地址空间，可以通过platform_device_add_data()进行绑定（绑定到dev->platform中）。调用关系如下：
+ platform_create_bundle
   + platform_device_add_properties
   
如果在创建平台设备时没有分配地址空间，例如acpi_platform.c在基于DSDT表注册平台设备时，就不会为platform_data创建地址空间，驱动可以自行创建platform_data，并绑定到dev->platform中。acpi之所以没有创建该数据，因为每一个acpi设备的platform_data数据结构都是自定义的，无法统一创建。

# 平台设备与平台驱动的绑定

# ACPI注册平台设备
ACPI驱动通过DSDT表里面的HID来识别是否为平台设备（type.platform_id=1）。ACPI会再DSDT表里面进行walk，为找到的每一个平台设备进行platform_device的注册：

apci_bus_scan
-> acpi_bus_attach
   -> acpi_default_enumeration
      -> acpi_create_platfrom_device.
      
DSDT表中，Device是有嵌套关系，acpi会根据嵌套关系以{父设备，子设备}的顺序依次申请

Device（GPO1）{
     Device（PRTA） {
     }
}

由于platfrom_device包含device，因此，这种父子关系会传递到device中。在device中，通过父子节点来表示这种层级，因此可以使用device_get_child_node_count(struct device * ）来获取子节点的个数，或者通过device_for_each_child_node（）来进行子节点的遍历。

