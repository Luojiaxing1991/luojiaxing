+ 简介
+ PHY layer的子功能（硬件协议）

# 简介
这篇报告主要以SCSI phy为中心进行分析，试图将phy类的框架及协议从SCSI这个复杂整体中剥离出来，维持phy上下层(接口->驱动->硬件)贯通性的同时，降低理解的难度。

phy的概念是一个物理接口，因此在phy这一层级，host，背板和硬盘都只被视为介质，并且只会考虑速率协商，Double word数据的成功传输这一类基本的phy功能，其他诸如硬盘地址，原语，甚至是更上层的命令帧，数据帧，对于phy而言，都被分解为了原子的一个character概念。phy保证character能够成功地发送到对端介质。

下面是协议对phy layer的简介，同样强调了phy主要用于做phy reset以及跟踪dword synchronization：
``` C
The phy layer defines 8b10b coding, BMC coding, and OOB signals. Phy layer state machines interface between the link layer and the physical layer to perform the phy reset sequence and keep track of dword syncronization
```

下面是scsi层对于phy的定义：结构体struct sas_phy{}，定义在include/scsi/scsi_transport_sas.h
``` C
struct sas_phy {
	struct device		dev;
	int			number;
	int			enabled;

	/* phy identification */
	struct sas_identify	identify;

	/* phy attributes */
	enum sas_linkrate	negotiated_linkrate;
	enum sas_linkrate	minimum_linkrate_hw;
	enum sas_linkrate	minimum_linkrate;
	enum sas_linkrate	maximum_linkrate_hw;
	enum sas_linkrate	maximum_linkrate;

	/* link error statistics */
	u32			invalid_dword_count;
	u32			running_disparity_error_count;
	u32			loss_of_dword_sync_count;
	u32			phy_reset_problem_count;

	/* for the list of phys belonging to a port */
	struct list_head	port_siblings;

	/* available to the lldd */
	void			*hostdata;
};
```
注：对于scsi层，并不负责phy的管理，上面定义的sas_phy结构体主要目的是提供一个通用的结构体定义给底层软件，方便信息透传。因此这个结构体的适用对象就是phy，无论是直连组网下的近端phy，还是expander组网下的近端phy或者远端phy，都可以用这个结构体。

从协议以及驱动结构体的定义分析，我们可以得出phy layer的两个基本功能：
+ phy layer需要负责两个介质之间的速率协商（主要通过 phy reset sequence实现）
+ phy layer需要保证两个介质之间的原子数据包（DW大小）的正常传递

# PHY layer的子功能（硬件协议）
根据SAS 3.0协议，PHY layer主要包括三个功能： 8b10b coding, BMC coding, and OOB signals。

## 8b10b coding
8b10b coding就是将8 bit的数据转译为10 bit的character发送出去。对于PHY，一个DW包括4个character。除了原始的8bit数据外，PHY追加了2个bit用于标识控制信息，发送原语（primitive）还是发送数据。

对于转译后的10 bit数据，会通过物理链路一位一位(bit-by-bit)地串行发送出去。

SAS协议提供了1024个可能的10 bit character，这些character可以涵盖【0~0xFF】范围内所有数字字符（Data character），另外也涵盖控制字符（control characters）,用于一些原语或者控制目的，其他的为无效字符。K（primitive）和D（data）的10 bit译码是区分开来的。具体可以查表 5.3.6 Data characters 和 5.3.7 Control characters。

8b10bit编码的目的主要是确保串行比特流中有足够的转换，从而使得在接收器处可以恢复时钟。另外还增加了在传输和接受期间检测单个或者多个比特错误的可能性。另外，一些控制字符会包含一个不同的位模式，称为逗号模式，这有助于接收方在传入的比特流上实现字符与Dword对齐。

### 8b10b coding notation
SCSI特定为character定义了命令规则（Zxx.y），辅助协议的可读性。例如，内容为DCh的一个原语，在SCSI协议中通过K28.5来标识：
+ 28 = DC[bit0:4]
+ 5 = DC[bit5:7]
+ K表示原语。

用户可以通过Zxx.y的命令在译码索引表中找到对应的10bit译码。

### transmission order
+ bit的传输顺序是基于10bit译码直接发送
+ character的传输顺序：如果是primitive，则一定会先传control character（例如k28.5）
+ frame的传输顺序：

由于8b10b的概念过于偏向于底层硬件，所以具体入一些位的传输顺序，字符传输顺序等，在这里不赘述，有兴趣的可以查看协议 5.3 Character encoding and decoding。

## Dword与原语
所有字符都是基于4个字符为一组的方式在SAS线中传输，被称为Dword。

原语是Dword中的一种，它固定以K28.3或者K28.5作为起点。其中SAS以K28.5（可以查看协议6.2.6或者6.2.7）为Dword中第一个字符，而SATA以K28.3或者K28.5(可以查看协议6.2.8)作为Dword第一个字符
。

## Out of band(OOB) signal
后续更新

## Phy capabilities bits
如果phy的两端支持SNW-3，在做Train_Tx-SNW之前，需要互相交换Phy capabilities bits。这是一个32bit的数据块。主要包括5块内容，1个起始位，一个TX SSC TYPE标记位，用于标记SSC（这个标记位的具体定义查看协议5.7 Phy capabilities bits），一个 REQUESTED LOGICAL LLINK RATE域，用于指示是否支持multiplexing；一个Supported Settings域，用于标识支持的速率，Train_Tx-SNW和Train_Rx-SNW会基于这个来生成训练的parameter，以及一个PARITY位，用于校验。
