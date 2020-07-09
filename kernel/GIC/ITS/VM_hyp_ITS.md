+ 目的


# 目的
研究VM，hypervisor与ITS之间的关系链条。理清楚vCPU调度链条。

# VM 与 irq_domain

# vcpu的调度与ITS
vcpu相当于是cpu上运行的一个进程，它的整体框架会遵循进程调度的基本逻辑，例如vcpu等待中断时会调用schedule（）进行主动调度，需要运行时会发起抢占，或者有时候被更高优先级进程抢占。

但是vcpu与普通进程有一个区别在于，它需要ITS配合，用于使能该vcpu的虚拟中断的接受（vgic_v4_put/vgic_v4_load）。ITS会配合上述两个函数，将vcpu对应的vPE在PE上进行调度。

这里存在一个问题。由于__schedule()函数包括当前进程的停止和下一进程的激活，一个函数做了两件事情。kvm_vgic_put/kvm_vgic_load分别嵌入到了sched_out/sched_in中。这是兼容框架的。但是，由于kvm_vgic_put默认调用vgic_v4_put(vcpu, false)，意味着ITS在vcpu deschedule时默认不会配置doorbell。这就使得doorbell永远不会被使用，这是有问题的。所以在kvm_arch_vcpu_blocking中（这里意味着vcpu需要等待中断而主动申请调度），在preempt_enable之前会调用vgic_v4_put(vcpu, true)来提前配置ITS。因为这种情况，doorbell是需要的。

## vcpu deschedule
vcpu deschedule一般包括下述两种情形：
+ vcpu等待中断或者时间片内完成工作，主动调度，例如下面的kvm_handle_wfx
``` C
kvm_handle_wfx - +  
                 | - kvm_vcou_block - +  
                                      | - kvm_arch_vcpu_blocking - +  
                                                                   | - vgic_v4_put(vcpu, true)  //true mean door bell is need

check_vcpu_request - + //put之后里面调用load，跟vPE deschedule无关，估计是一些reload操作  
                     | - vgic_v4_put(vcpu, false)  
                     | - vgic_v4_load  
                     
                     
kvm_arch_vcpu_ioctl_run - + //kvm 运行的主程序，当vcpu运行完毕后会正常退出。                    
                          | - vcpu_put - +  
                                         | - kvm_arch_vcpu_put   //rest is the same with sched_out  

kvm_preempt_ops.sched_out - + //当前vCPU被其他进程或者vcpu抢占  
                            | - kvm_sched_out - +  
                                                | - kvm_arch_vcpu_put - +  
                                                                        | - kvm_vgic_put - +  
                                                                                           | - vgic_v4_put(vcpu, false) - + //false mean that no door bell need  
                                                                                                                          | - itc_make_vpe_non_resident
```

## vcpu schedule
``` C
kvm_arch_vcpu_ioctl_run - +
                          | - vcpu_load - +
                                          | - kvm_arch_vcpu_load

kvm_sched_in - +
               | - kvm_arch_vcpu_load - +
                                        | - kvm_vgic_load - +
                                                            | - vgic_v3_load - +
                                                                               | - vgic_v4_load - +
                                                                                                  | - its_make_vpe_resident
                                                            

```

# vCPU的schedule/deschedule机制
当vCPU空闲或者等待物理内核处理一些事情的时候，vCPU会被调度出去，类似于进程调度，但是对于vCPU而言，并不会有阻塞CPU的行为，因为其他vCPU可能正在等待调度，因此正好可以把CPU的资源让渡出来。
KVM中提供了一个函数 handle_exit(struct kvm_vcpu *vcpu...),专门用于处理vcpu的退出机制。

KVM中有一个exit_handle_fn的结构体，定义了各种退出信号的处理，其中和vCPU block/unblock相关的主要是 ESR_ELx_EC_WFx，用于处理WFI和WFE两类信号（kvm_handle_wfx（））。

## doorbell发给了谁？
doorbell是一种中断，所以它只能发送给CPU，而CPU知道这个是doorbell中断，应该会通过回调函数的方式，进入hypervisor，从而让目标vCPU得到调度。在vCPU被调度的时候，GICR也需要将vPE调度到PE上。

## sched_in



