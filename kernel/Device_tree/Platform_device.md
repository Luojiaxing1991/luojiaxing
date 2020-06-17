+ 平台设备简介
+ 平台设备与平台驱动的绑定
+ struct platform_device
+ module_platform_driver
   + struct platform_driver
   + platform_data

# 平台设备简介

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

