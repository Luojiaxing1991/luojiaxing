+ MAPC


# MAPC
MAPC命令用于非虚拟机，主要作用是在对应的collection table entry中更新GICR的地址(index为ICID，基于cpu id而来)。当ITS执行完MAPC后，以ICID（cpu id）为index，通过ct表是可以知道GICR的地址，将某几个irq号和ICID做好mapping后，这几个irq就可以顺利的发送到指定的GICR。cpu在启动的过程中会指示ITS发送MAPC命令，将与自己连接的GICR地址更新到CT表。
注意：CPU需要通知每一个ITS都在各自的collection table的entry中更新GICR的地址。

``` C
ITS 0 ----MAPC（cpu 0）---> collection table entry 0 - +
                                                       | -- GICR 0
ITS 1 ----MAPC（cpu 0）---> collection table entry 0 - +
```

在ITS初始化的过程中，gic_smp_init（）通过cpuhp_setup_state_nocalls（）在cpu热插拔中注册了一个钩子函数，gic_starting_cpu(),这样cpu在初始化的时候回调用gic_starting_cpu()，其中有一个步骤是发送MAPC命令,每一个ITS都需要发送MAPC。

``` C
gic_strating_cpu - +
                   | - its_cpu_init - +
                                      | - its_cpu_init_collections - +
                                                                     | - its_cpu_init_collection - +
                                                                                                   | - its_send_mapc
```
