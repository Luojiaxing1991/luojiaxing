+ MAPC
+ INV/INVALL

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

# VINVALL
VINVALL命令用于保证GICR中对应vPE的vLPI configuration table内容与内存内容的一致性。vLPI configuration table是GICR共享的表，因此在GICR中有cache保存跟自身相关的vIRQ的配置信息。

VINVALL命令是由虚拟机中的用户发送GITS_CMD_INVALL命令后透传下来的。虚拟机的用户认为自己实际上是一台服务器，所以它下发的是INVALL命令。但透传下来是一个VINVALL命令。

INVALL命令在cpu初始化，或者cpu热拔插的时候会被执行。具体查看INVALL命令。
``` C
vgic_its_cmd_handle_invall - +
                             | - its_invall_vpe - +
                                                  | - its_send_vpe_cmd - +
                                                                         | - irq_set_vcpu_affinity - +
                                                                                                     | - its_vpe_4_1_set_vcpu_affinity - +
                                                                                                                                         | - its_vpe_4_1_invall
```
