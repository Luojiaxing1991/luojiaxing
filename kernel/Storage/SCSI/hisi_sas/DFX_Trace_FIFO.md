+ 需求分析

# 需求分析
DFX trace FIFO主要用于辅助芯片定位链路异常或者链路优化的DFX工具。DFX dump工具主要面向异常场景，而本工具主要面向非异常场景。因此，本工具与DFX dump工具的代码实现是隔离的。

从用户角度看，用户主要通过debugfs来使用DFX trace FIFO。hisi_sas DFX的基目录位于/sys/kernel/debug/hisi_sas/。我们通过dev_name(dev)在里面为每一个SAS控制器建立了一个文件夹。名字为PCIe总线号，例如0000:74:02.0。FIFO会在该文件夹下有一个独立的子文件夹，命名为trace_fifo。该文件夹不受trigger_dump文件的影响，这个文件只影响dump文件夹，属于DFX dump工具。

DFX trace FIFO有一些配置项可以提供给用户。
+ dump_disable
```
由于硬件设计DFX trace FIFO是按默认配置使能，并会实时往硬件cache里面写入状态数据。所以，在修改配置或者读取数据时，需要暂停FIFO的数据写入。硬件cache的大小为32 * 4byte = 128byte，每4个byte为一组完整数据。由于是实时写入，cache的数据是不断更新的，用户可以配置硬件在特定条件下，停止写入，或者继续写入32byte后停止。具体查看dump_mode。

dump_disable系统启动默认为0，就是说系统启动后硬件cache就会被不断写值。
```
+ signal_sel
```
trace_FIFO有不同的信号组，每组信号组的rd寄存器域值的含义不同，这个配置决定了用户可以看到的数据内容。

对于用户而言，选择不同的信号组，在查看rd数据时候应该提供不同的域值解析。
```
+ dump_msk
