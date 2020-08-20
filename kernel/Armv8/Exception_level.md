+ 简介
+ SCR_EL3

# 简介
EL(Exception Levle)是Armv8中首次引入的概念，每一个EL代表了不同的特权级别。

```C
    Normal world    |   Secure world
EL0 Application     |   Secure firmware
EL1 Guset OS,Kernel |   Trusted OS
EL2 Hypervisor      |   NA
EL3          Secure Monitor
```

特权等级随着ELx中x的增加而加大。

每个Exception Level的意义如下：
+ EL0: 用户控件，在Normal world中是指运行的应用程序，或者虚拟机，在Secure World就是TA，Trust Application
+ EL1：运行操作系统，在Normal world中是指linux操作系统，在Secure World就是Trusted OS，鸿蒙OS的前身就是安全OS
+ EL2：ARM为了支持虚拟化，设计的Hypervisor层。Armv8只有Normal world使用。
+ EL3：Secure Moniter的作用是用于Normal world和Secure world切换使用。当Normal world想要访问secure world时，需要发送SMC指令进入Secure Moniter层然后进入Secure world

# SCR_EL3
SCR_EL3（Secure）这个系统寄存器用于Secure configuration。具体可以查看系统寄存器手册。

