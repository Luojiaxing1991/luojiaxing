+ 需求分析

# 需求分析
DFX trace FIFO主要用于辅助芯片定位链路异常或者链路优化的DFX工具。DFX dump工具主要面向异常场景，而本工具主要面向非异常场景。因此，本工具与DFX dump工具的代码实现是隔离的。

从用户角度看，用户主要通过debugfs来使用DFX trace FIFO。hisi_sas DFX的基目录位于/sys/kernel/debug/hisi_sas/。我们通过dev_name(dev)在里面为每一个SAS控制器建立了一个文件夹。名字为PCIe总线号，例如0000:74:02.0。FIFO会在该文件夹下有一个独立的子文件夹，命名为trace_fifo。该文件夹不受trigger_dump文件的影响，这个文件只影响dump文件夹，属于DFX dump工具。

## 配置项
DFX trace FIFO有一些配置项可以提供给用户。
+ dump_disable
```
由于硬件设计DFX trace FIFO是按默认配置使能，并会实时往硬件cache里面写入状态数据。所以，在修改配置或者读取数据时，需要暂停FIFO的数据写入。硬件cache的大小为32 * 4byte = 128byte，每4个byte为一组完整数据。由于是实时写入，cache的数据是不断更新的，用户可以配置硬件在特定条件下，停止写入，或者继续写入32byte后停止。具体查看dump_mode。

dump_disable系统启动默认为0，就是说系统启动后硬件cache就会被不断写值。
除了0和1以外，不接受其他设值。
```
+ signal_sel
```
trace_FIFO有不同的信号组，每组信号组的rd寄存器域值的含义不同，这个配置决定了用户可以看到的数据内容。

对于用户而言，选择不同的信号组，在查看rd数据时候应该提供不同的域值解析。
signal_sel的输入值范围参考寄存器，如下：{0x0, 0x1, 0x10, 0x11, 0x100, 0x101},定义参考寄存器手册。
0x0: 速率协商打包信号组 (默认配置)
0x1: 12G tx train本端训练对端的调整方式，以及与HiLink的交互信号
0x10: 12G tx train对端训练本端的调整方式
0x11: sas_phy_ctrl_dfx0
0x100: sas_phy_ctrl_dfx1
0x101: sas_pcs_dfx0
others: 速率协商打包信号组
```
+ dump_msk
```

dump_msk的输入值范围：{0x~0xFFFFFFFF}
```
+ dump_mode
```
dump_msk的输入值范围：{0x1,0x10,0x100},具体定义参考寄存器手册。
0x1: 一直dump
0x10: 向前dump，即触发条件触发后，再dump32个数据就停止
0x100: 向后dump，即触发条件触发后，不在dump DFX
```
+ trigger_mode
```
trigger_mode的输入值范围：{0x1,0x10,0x100},具体定义参考寄存器手册。
0x1: 当cfg_dfx_trigger_msk对应的bit为1的DFX信号发生跳变的时候触发
0x10: 当DFX信号等于cfg_dfx_trigger(cfg_dfx_trigger_msk对应的bit为1)时触发
0x100: 当DFX信号不等于cfg_dfx_trigger(cfg_dfx_trigger_msk对应的bit为1)时触发
```
+ trigger/trigger_msk
```

trigger/trigger_msk的输入值范围：{0x~0xFFFFFFFF}
```
DFX trace FIFO配置接口的默认值从系统启动后的对应寄存器中获取。仅root用户可以读写。debugfs对输入有限制的参考之前配置项需求描述细节。

## 数据读取
用户可以通过data的文件来查看本次trace出来的DFX数据。该文件仅root用户可读。在文件的开头，标识出本次trace的配置数据一览。然后依次解析32组dump芯片状态数据包。



