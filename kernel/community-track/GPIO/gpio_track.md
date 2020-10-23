+ Rethinking struct gpiod_line_bulk
+ gpio bias概念研究
+ Introduce the for_each_set_clump marco
+ gpiolib: cdev: allow edge event timestamps to be configured as REALTIME
+ tools: gpio研究
+ Respect bias settings for GpioInt() resource
+ GPIO的acpi研究


# Rethinking struct gpiod_line_bulk
这是一个构建libgpiod v2.0的一个改善点。

# gpio bias概念研究
bias这个概念在linaro connect也提及，但是我对这个并不熟悉，可以研究下

# Introduce the for_each_set_clump marco
这是一个kernel级别的patch，提供一个轮训，可以对几组数据（8bit一组）中的一组单独更新？

# gpiolib: cdev: allow edge event timestamps to be configured as REALTIME

# tools: gpio研究
linaro connect中提及的uAPI代码也涉及tools: gpio的修改，这个不是非常清晰

# Respect bias settings for GpioInt() resource
使用GPIO PIN作为中断源的模块会通过acpi_dev_gpio_irq_get来获取GPIO PIN对应的linux irq中断号。看作者的commit是指，如果BIOS的GpioInt对bias有配置的话，优先使用BIOS的配置。
但是我对GPIO如何获取BIOS资源这一块的细节还不熟悉（BIOS的那些配置会作为GPIO的配置保留下来，这一块并不熟悉），需要新增一个研究点。

# GPIO的acpi研究
之前有部分初步研究，待深入
