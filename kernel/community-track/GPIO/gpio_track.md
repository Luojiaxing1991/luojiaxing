+ Rethinking struct gpiod_line_bulk
+ gpio bias概念研究
+ Introduce the for_each_set_clump marco
+ gpiolib: cdev: allow edge event timestamps to be configured as REALTIME
+ tools: gpio研究


# Rethinking struct gpiod_line_bulk
这是一个构建libgpiod v2.0的一个改善点。

# gpio bias概念研究
bias这个概念在linaro connect也提及，但是我对这个并不熟悉，可以研究下

# Introduce the for_each_set_clump marco
这是一个kernel级别的patch，提供一个轮训，可以对几组数据（8bit一组）中的一组单独更新？

# gpiolib: cdev: allow edge event timestamps to be configured as REALTIME

# tools: gpio研究
linaro connect中提及的uAPI代码也涉及tools: gpio的修改，这个不是非常清晰

