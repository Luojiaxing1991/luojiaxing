
gic_smp_init（）通过cpuhp_setup_state_nocalls（）在cpu热插拔中注册了一个钩子函数，gic_starting_cpu(),这样cpu在插入后

``` C
gic_strating_cpu

gic_init_bases


its_cpu_init

```
