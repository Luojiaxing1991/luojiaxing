+ GPIO简介
+ 驱动如何获得GPIO硬件描述
+ 用户如何获得GPIO硬件资源
   + GPIO desc
      + GpioINT
      + GpioIO
   + GPIO PIN interrupt
+ GPIO框架
+ 在SYSFS里面调试GPIO
   + 获得一个GPIO口进行调试


# GPIO简介
GPIO的内核描述文档位于/Documentation/driver-api/gpio.

# 驱动如何获得GPIO硬件描述

# GPIO控制器中断获取

# 用户如何获得GPIO硬件资源

## GPIO desc
用户可以通过gpiod_get_index()来或者一个GPIO的desc。

gpiod_get_index(struct device *dev, const char *con_id, unsigned int idx, enum gpiod_flags flags)

这个API包括四个入参，dev主要指GPIO的消费者，con_id描述这个消费者对这个GPIO的用途（自定义），idx描述在这个用途下的GPIO的index，flag。

这些资源一般会在ACPI的DSDT表或者设备树资源描述里，以ACPI举例：
``` C
Device(SPI0){
  Name(_DSD, Package() {“cs-gpios”， Package(){^SPI0, 0, 0, 0,},},)
}
```

一般ACPI会把消费者（用户模块）对于GPIO的描述放在_DSD里面。通常以package宏来定义。我们可以这样解释上面这个描述：SPI0有一个cs-gpios的功能是通过{^SPI0,0,0,0}的GPIO口实现的。gpiod_get_index()中的con_id其实就对应“cs”,由于gpiolib要求ACPI添加GPIO特定描述，因此，ACPI在扫描GPIO资源时会组合 {con_id}-gpios进行组合（参考acpi_find_gpio()）。

如果_DSD中也找不到该GPIO资源，则会尝试在_CRS里面查找package。

另外，Package相当于C语言中的数组，而idx指的是con_id（在上面的例子里面就是cs-gpios）对应的Package, 由于里面只有一个数组{^SPI0,0,0,0}，那么只有idx=0的时候能返回有效值。

驱动会通过__acpi_node_get_property_reference()获取到con_id的目标package，当然现在还是最原始的状态，相当于一个数组。里面的信息是有具体模块或者用途来解析的。比如GPIO的话，在acpi_gpio_property_lookup()会进行翻译和信息分门别类的保存操作。举个例子，{^SPI0,0,0,0}是被如下解析的。

整数则表示有实际意义的标识，而字符串则是定义为local reference。  {REF,0,0,0}.其中 local referenc会被解析为一个acpi_device，并通过acpi_fwnode_handle获取fwnode_handle保存在struct fwnode_reference_args{}。 而其他整数部分会保存在args[]。

struct fwnode_reference_args{
  struct fwnode_handle *fwnode;  //local reference
  u64 args[];  //package的整数部分
}

通过上述获取动作，我们拿到GPIO消费者依赖的一个设备的fwnode，然后可以通过acpi_dev_get_resources()调用acpi_populate_gpio_lookup

### acpi_resource_gpio
当acpi在扫描DSDT表时，一个device里面的描述会保存到这个文件夹，例如如果device有GpioINT或者GpioIO的话，agpio里面的connection_type会被设置为TYPE_INT或者TYPE_IO.另外还有polarity，triggering等信息也是同理

### GpioINT
acpi_populate_gpio_lookup（）会区分GpioINT和GpioIO，根据

### GpioIO

### acpi_gpio_info

fwnode_get_named_gpiod()会通过 acpi_node_get_gpiod（）获取gpio_desc和 struct acpi_gpio_info，然后通过info来设置一些gpio_desc的flags。包括一些中断的极性，IO PIN口的特性等。

综上，用户通过gpiod_get_index()获取到gpio_desc后，所有配置在BIOS的该GPIO口的硬件资源都获取到了。


## GPIO PIN interrupt

# 在SYSFS里面调试GPIO
编译内核时，通过make menuconfig中勾选Device Drivers->GPIO Support->/sys/class/gpio/...(sysfs interface)。

系统中可以找到/sys/class/gpio目录。

## 获得一个GPIO口进行调试
gpio目录下有三类文件，export,unexport,gpiochixxx文件夹。用户可以通过export获取一个GPIO口的控制，unexport来释放。对用户可将的GPIO控制器信息由gpiochipxxx来获得。

### gpiochipxxx
gpiochipxxx包含一些GPIO控制器的属性，其中base是当前GPIO控制器第一个GPIO口的偏移。用于export/unexport。而ngpio则标记了这个GPIO控制器的GPIO口数量。

``` C
echo 480 > export  //480 = base + offset
```

### gpioxxx
当echo 480 > export后，gpio文件夹中会新增一个gpio480的文件夹。
