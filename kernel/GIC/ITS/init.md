# ITS初始化的组成
- ITS本身的初始化
- 为MSI/platform设备建立irq_domain
- 每个CPU online时进行对应的ITS初始化

# ITS初始化入口

gic_acpi_init()调用gic_init_bases()；
如果ITS的内核编译选项打开，且硬件支持ITS，则gic_init_bases()会调用 its_init() 和 its_cpu_init()执行ITS的初始化。

用户可以在内核启动参数中 配置 irqchip.gicv3_nolpi=0 ，来禁止ITS的初始化。

其中，its_init()用于ITS的软件初始化，而its_cpu_init()主要用于CPU0的ITS初始化，由于CPU0在online的时候，ITS还未加载，没有将its_cpu_init()这个函数注册到cpu的启动流程中，所以CPU0的ITS初始化需要手动调用。

# ITS硬件设备扫描
BIOS上报了2个ITS设备，分别在NUMA0和NUMA2上，具体信息分别保存在SRAT和MADT表中,表的index分别为 ACPI_SRAT_TYPE_GIC_ITS_AFFINITY 和 ACPI_MADT_TYPE_GENERIC_TRANSLATOR

its_init()最开始会扫描apci的SRAT和MADT表，ACPItable的扫描接口函数是acpi_table_parse_entries()，通过第一个入参来指定表的类型。

## SRAT表
SRAT表里面信息不多，主要就是its_id，以及这个its的proximity信息，proximity信息其实就是标示这个ITS与周边NUMA节点的距离用的，所以通过acpi_map_pxm_to_node()可以找到这个ITS的numa节点信息
可以看看gic_acpi_parse_srat_its(),来获取具体代码，最终目的是把node节点和its_id保存到its_srat_maps[]数组中，注意，这个数组的index和its_id并不一样。

## MADT表
MADT表里面比较重要的信息有两个,一个是translation_id，其实跟SRAT表里面的its_id是相同的值，一个是base_address，这个是一个物理地址，ITS自身的IORESOURCE_MEM的起始地址，然后这一段地址的长度是固定的 SZ_128K。

得到MADT表的信息后，ITS开始在irq_domain中注册了一个fwnode，并且在ACPI的IORT表（IO remapping table）中为这个fwnode注册了一个token，注释说，在msi_chip的list里面把ITS加进去了，方便后续回调获取base_address信息。但暂时不明其用途
fwnode注册完毕后，可以对ITS设备进行probe操作。

## its_probe_one
判断GIC版本，通过寄存器 GITS_PIDR2 判断当前的GIC版本是否为v3或者v4，如果不是，probe终止

停止ITS硬件以便执行初始化，通过GITS_CTLR的GITS_CTLR_ENABLE对ITS执行disbale命令，并通过GITS_CTLR的GITS_CTLR_QUIESCENT确认ITS工作完全停止

填充ITS的基础信息

如果当前GIC版本是v4版本，那么还需要把当前的ITS添加到its_list_map，这是为VMOVP命令做准备

为cmd queue准备内存空间，同时，指定its_irq_get_msi_base为接口函数，其他模块可以通过该接口获取GITS_TRANSLATER寄存器的地址，用来写入当前中断的eventID。deviceID写入ITS command中。

its_alloc_tables比较复杂，可以单独起一个小标题来讲解

## its_alloc_tables
这个函数需要依次为8个baser创建对应的table（如果是L2 table，则需要创建L1 table），每一个baser指向的table 类型已经预先被硬件固定了，驱动根据不同的table类型做相应的处理。这个函数主要需要为DT表和VPE表进行L2的判断，如果是CT表，则直接分配64K空间。

### 为DT和VPE表设计table(level1 or level2)
调用 its_parse_indirect_baser() 为DT表和VPE表需要在这里进行L1，L2的表格设计，由于entry size（esz）和 entry个数（入参：ids）已知，那么可以根据 内存空间大小来决定是否使用L2table，如果 memory requirement < (psz*2)bytes，那么直接使用L1table，否则使用L2table
L2table的大小固定为 ITS page size(64K), 那就是说 64K空间中能够存储 N 个 entry，如果总数M，能切分为 X 个 N组，那么 L1 table里面至少需要存放X个entry

L1table的 entry主要存放valid标记和L2table的物理首地址，而且L2 table首地址是以ITS PAGE size 对齐，也就是说 第N位都是0（N取决于ITS Page size）

its_parse_indirect_baser（）主要返回是否需要L2table（通过返回值 indirect 标记），以及L1table的大小（通过回写入参 order决定）

另外，VPE目前最大支持2^16个，而Device的个数则有寄存器 GITS_TYPER 中指定

执行alloc_tables()后，调用its_alloc_collections开始collection表的内存分配，这个collection表内部维持collection_id与GICR的phy_address的一一对应关系。但是当前只是分配内存空间，直接赋初始值。
这个collection跟baser里面指向的CT表不通，baser那个CT表是标注IRQ id与collection id之间的一一对应关系的。这里是collection id与GICR之间的对应关系。

为CMD queue分配内存空间，并更新写指针为0

调用its_init_domain 申请了一个domain，不过暂不理解这个domain的粒度。

至此， ITS 设备的probe操作就完成了，its_acpi_probe()函数返回，继续执行 its_init的剩余初始化操作。

## allocate_lpi_tables()
LPI有两个table，一个是LPI Configuration table，一个是LPI Pending table。

- LPI Configuration table: 每一个INTID都有都有一个byte来记录它的优先级
- LPI pending table：每一个INTID都有都有1bit来记录它的状态

每一个CPU都有一个pending table，pending table中的每一个位对应一个lpi中断


最后通过 register_syscore_ops 注册一个ITS的syscore_ops钩子函数到syscore，当CPU online后，需要通过这个钩子函数往ITS里面注册该CPU的GICR信息


its_init()的初始化流程到此结束。

接着继续 its_cpu_init()

这个初始化应该是每一个CPU起来后都需要做的，但是由于ITS初始化时是CPU0，他是第一个CPU，他起来的时候GIC模块的代码还没有跑，所以CPU0的its_cpu_init()在its初始化后会被调用一次，为CPU0服务。
其他CPU在 start up流程中会调用its_cpu_init（）

gic_smp_init
-》 cpuhp_setup_state_nocalls
-》 gic_starting_cpu
  
通过 cpuhp_setup_state_nocalls 可以在cpu hostplug ops里面注册一个 钩子函数， 这里是把 its_cpu_init()注册为了让cpu startup的时候可以去调用这个钩子函数

Installs the callback functions and invokes the startup callback on the present cpus which have already reached the state.

https://www.kernel.org/doc/html/v4.11/core-api/cpu_hotplug.html


## its_cpu_init

这个函数主要调用两个函数 its_cpu_init_lpis() 以及 its_cpu_init_collections()

### its_cpu_init_lpis
主要是 获取prop_table，记录在GICR的寄存器中，从内存分配 pending_table，记录在GICR的寄存器中

### its_cpu_init_collections
获取对应CPU的id作为col id，以及该CPU对应的GICR的physical address，并发送命令给ITS，让ITS建立对应的col id与GICR的映射关系。

#### MAPC
