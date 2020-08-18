+ qemu-kvm简介
   + qemu介绍
   + kvm介绍
   + qemu-kvm
+ 基本原理
+ 参考网页及资料

# qemu-kvm简介

## qemu介绍
QEMU(short for Quick Emulator)是一种提供硬件虚拟化的免费开源主机hypervisor。QEMU是一个主机上的VMM（Virtual machine monitor）,它可以通过动态二进制来模拟CPU，或者利用KVM，XEN等虚拟化技术来模拟CPU，内存；另外，它也提供了一系列的硬件模型（硬盘，网卡等），是guest os认为自己和硬件直接打交道，其实是在和QEMU模拟出来的硬件打交道，而QEMU再次将这些指令翻译给真实的硬件进行操作。

QEMU其实只是一个硬件虚拟器，但是如果让QEMU去模拟一个完整的操作系统（它支持多种guest OS：linux，windows等），它的性能会比较差。

## kvm介绍

## qemu-kvm


# 基本原理
QEMU作为系统模拟器时，会模拟出一台能够独立运行操作系统的虚拟机。每一个虚拟机对应主机中的一个QEMU进程，而虚拟机的vCPU对应QEMU进程的一个线程。

系统虚拟化最主要是虚拟出CPU、内存与I/O设备。虚拟出的CPU称之为vCPU，QEMU为了提高效率，借用KVM，XEN等虚拟化技术，直接利用硬件对虚拟化的支持，在主机上安全地运行虚拟机。虚拟机vCPU调用KVM的接口来执行任务的流程如下：

``` C
//下面的例子是以KVM
open("/dev/kvm")
ioctl(KVM_CREATE_VM)
ioctl(KVM_CREATE_VCPU)
for (;;) {
ioctl(KVM_RUN)

  switch (exit_reason) {
    case KVM_EXIT_IO: /* ... */
    case KVM_EXIT_HLT: /* ... */
  }
}
```

上面这个例子非常经典的介绍了qemu使用KVM的例子。通过sysfs打开dev/kvm，创建一个VM，为VM创建VCPU，运行VM这个进程，当有IO完成或者中断，则中断进程，通过KVM获取到数据后再重启进程运行。

QEMU发起ioctl来调用KVM接口，KVM则利用硬件扩展直接将虚拟机代码运行与主机之上，一旦vCPU需要操作设备寄存器，vCPU就会停止并返回QEMU，QEMU去模拟出操作结果。

虚拟机内存会被映射到QEMU的进程地址空间，在启动时分配。在虚拟机看来，QEMU所分配的主机上的虚拟地址空间为虚拟机的物理地址空间。

QEMU在主机用户态模拟虚拟机的硬件设备，vCPU对硬件的操作结果会在用户态进行模拟。如果虚拟机需要将数据写入硬盘，实际结果是将数据写入到了主机中的一个镜像文件中。

# 参考网页及资料

## wiki of qemu
wiki.qemu.org

## qemu 官方文档
https://www.qemu.org/docs/master/system/quickstart.html
