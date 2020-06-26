+ 目的


# 目的
研究VM，hypervisor与ITS之间的关系链条。理清楚vCPU调度链条。

# VM 与 irq_domain


# vPE deschedule

kvm_handle_wfx - +  
                 | - kvm_vcou_block - +  
                                      | - kvm_arch_vcpu_blocking - +  
                                                                   | - vgic_v4_put(vcpu, true)  //true mean door bell is need

check_vcpu_request - + //put之后里面调用load，跟vPE deschedule无关，估计是一些reload操作  
                     | - vgic_v4_put(vcpu, false)  
                     | - vgic_v4_load  
                     
                     
kvm_arch_vcpu_ioctl_run - + //vCPU没能在规定的时间片内完成所有操作，被调度出去后需要立即进入排队。                    
                          | - vcpu_put - +  
                                         | - kvm_arch_vcpu_put   //rest is the same with sched_out  

kvm_preempt_ops.sched_out - + //vCPU抢占  
                            | - kvm_sched_out - +  
                                                | - kvm_arch_vcpu_put - +  
                                                                        | - kvm_vgic_put - +  
                                                                                           | - vgic_v4_put(vcpu, false) - + //false mean that no door bell need  
                                                                                                                          | - itc_make_vpe_non_resident


# vPE schedule

kvm_arch_vcpu_unblocking

# vCPU的schedule/deschedule机制
当vCPU空闲或者等待物理内核处理一些事情的时候，vCPU会被调度出去，类似于进程调度，但是对于vCPU而言，并不会有阻塞CPU的行为，因为其他vCPU可能正在等待调度，因此正好可以把CPU的资源让渡出来。
KVM中提供了一个函数 handle_exit(struct kvm_vcpu *vcpu...),专门用于处理vcpu的退出机制。

KVM中有一个exit_handle_fn的结构体，定义了各种退出信号的处理，其中和vCPU block/unblock相关的主要是 ESR_ELx_EC_WFx，用于处理WFI和WFE两类信号（kvm_handle_wfx（））。

## doorbell使能与否




