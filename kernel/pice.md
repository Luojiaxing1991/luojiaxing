# MSI

## MSI与MSI-X的区别
一个只支持MSI的PCI设备最多支持 32个 IRQ申请，因为 BAR空间里面 存储 IRQ状态的位 只有 5bits
MSI-X则支持 2048 个 IRQ

## 亲和性MSI申请

通过pci_alloc_irq_vectors_affinity（）进行中断分配的申请，相对于正常申请中断用的 pci_alloc_irq_vectors（） 多了一个入参：struct irq_affinity desc;

这个结构体的构成如下：
struct irq_affinity {
	unsigned int	pre_vectors;
	unsigned int	post_vectors;
	unsigned int	nr_sets;
	unsigned int	set_size[IRQ_AFFINITY_MAX_SETS];
	void		(*calc_sets)(struct irq_affinity *, unsigned int nvecs);
	void		*priv;
};

pre_vectors和post_vectors是一个用来定义 前多少个IRQ不需要亲和性和后多少个IRQ不需要亲和性

nr_sets 和 set_size为我们提供了一种带亲和性的interrupt set，意思就是 带亲和性的这些IRQ中，可以用nr_sets指定里面有多少组interrupt set，每组set里面各自包含多少IRQ，IRQ_AFFINITY_MAX_SETS规定sets的上限为4
		注意，这两个参数不能直接赋值，要通过 calc_sets 进行赋值操作，如果 calc_sets 为空， 那么在 pci_alloc_irq_vectors_affinity函数中 默认 nr_sets 为1，set_size【0】为 可支持的 亲和IRQ个数

注意： pre_vectors + post_vectors >= pci_alloc_irq_vectors_affinity入参：minvec

## 申请亲和性中断
pci_alloc_irq_vectors_affinity的返回值： 成功的情况就是本次 申请到的IRQ个数，如果失败，则返回错误码。

用这个函数申请亲和IRQ时，能申请的亲和IRQ个数取决于 
		1.活动CPU的个数，即 亲和IRQ个数 <= online CPU个数。
		2.该设备支持的IRQ个数上限，MSI为 32，MSI-X为 2048
    
由于PCI是通过ITS来分配IRQ，PCIE中就MSI有一个order的约束，即 分配的IRQ个数必须是2^order的个数。但是这个约束由于是对于IRQ分配而言的，所以在ITS模块中，
分配IRQ会遵循这个order原则，同时释放的时候也是如此，但是在msi.c中并不考虑这个约束，而是按照实际的申请数来给ITS入参。

如果调用 pci_alloc_irq_vectors_affinity ， 入参 flags必须附上 PCI_IRQ_AFFINITY 。如果没有该flag，入参affd没有效果，会被置0； 如果附上该flag，
但是没有配置affd，默认affd中所有元素为 0，即为所有IRQ附带亲和性。

PCI_IRQ_LEGACY 这个flag什么意思？没怎理解到

pci_alloc_irq_vectors_affinity
	1.判断 PCI_IRQ_AFFINITY 是否置位，否，则此函数效果同 pci_alloc_irq_vectors ，是， 则检查affd，默认不设值，则affd元素全0，申请的所有IRQ均带亲和性。
	2.分别处理 MSI-X 和 MSI 中断的申请
	3.对于 PCI_IRQ_LEGACY 被置位的情况，也有特殊处理，不过不知道这个干什么用的。
-》__pci_enable_msi_range
	1.简单检查 入参  minvec 和 maxvec 的设值，并从 PCI bar空间读取该设备最多支持的 IRQ个数 nvec ， MSI设备最多是32个. 如果 nvec < minvec，说明MSI无法支持这么多的IRQ个数,返回错误。 
	  最后用min(nvec,maxvec) 作为本次申请中断的总个数（包括亲和和非亲和的IRQ），如果申请超过MSI所支持IRQ个数的中断，那么只能申请 MSI所支持IRQ个数 的中断。
	2.对于affd 非空的情况，会调用 irq_calc_affinity_vectors 来计算优化后的 中断申请总数 ；  这个中断申请总数 就是 pci_alloc_irq_vectors_affinity的返回值，后续的操作能保证该总数的IRQ被申请到，如果申请不了，会返回错误码，如果是IRQ没有分配完，会有calltrace
	   -》 irq_calc_affinity_vectors 
		a. 首先， 非亲和中断的个数，必须 小于等于 affd中 pre_vectors+post_vectors的值，否则 直接返回失败
		b. 如果 calc_sets 函数指针非空，说明 亲和IRQ中存在若干组，每组IRQ都能能对应所有CPU，所以没法根据possible CPU来决定 亲和IRQ 的个数，因此，用 nvec - 非亲和IRQ个数，作为本次申请的 亲和IRQ总数。至于能否成功，由 calc_sets 来保证。
		   -》这种情况下，这个函数返回的其实就是入参值 maxvec 。 calc_sets需要保证它的分组策略，能否满足 在CPU数量较少的情况下，也能申请成功。
		c. 如果 calc_sets 函数指针为空，说明 只有一组 亲和IRQ，那么，亲和IRQ的申请总数 不可以 大于 possible CPU的个数。
		   -》这种情况下，返回值是 非亲和IRQ个数 + min（possible CPU个数， 亲和IRQ个数）
	3.用 计算好的 中断申请总数 开始申请中断（调用 msi_capability_init ）。
    -》 msi_capability_init
	1.调用 msi_setup_entry 为PCI设备创建一个 struct msi_desc *entry; 用来管理这个 设备下的所有 IRQ
     -》 msi_setup_entry
     	1. 为每一个亲和IRQ创建 亲和mask
           -》irq_create_affinity_masks
		1. 根据 中断申请总数 除开 非亲和IRQ个数， 得到亲和IRQ个数
		2. 调用 calc_sets， 为 nr_sets 和 set_size 数组 赋值，默认 nr_sets为1， set_size【0】为 亲和IRQ个数。就是说 默认亲和IRQ只有一个分组，所有亲和IRQ都在这个分组
		3. 首先，将 中断申请总数中 头部的 非亲和IRQ的 mask 设置为 irq_default_affinity， 就是说 可以绑定到 任何 活动的CPU上面。
		4. 第二步，开始处理 亲和IRQ，根据亲和IRQ组的个数，用for循环依次为每组亲和IRQ进行mask的设值（ irq_build_affinity_masks ）
		   -》 irq_build_affinity_masks
			1. 获取每一个活动的CPU，根据这个CPU属于哪个node，把CPU的bit添加到该nodemask的bitmap中， 并最终形成一个 node_to_cpumask 数组，里面保存有每一个node的活动CPU的mask
			2. 调用 __irq_build_affinity_masks ，将这些活动CPU 添加到 每个亲和IRQ 的mask里面
			 -》 __irq_build_affinity_masks    中断亲和性策略
				1. 活动CPU个数为 0， 返回0
				2. 情况1： 申请的 亲和IRQ个数 <= node个数， 把第一个node的mask 拷贝给 第一个 IRQ, 依次处理，如果 node 有余， 那么再从头从第一个IRQ开始，继续追加 node的mask
					   就是说一个node上面的所有CPU都在 一个IRQ的mask上，而且，有一个或者几个IRQ甚至拥有多于一个的node可以处理其中断。
				3. 情况2： 申请的 亲和IRQ个数 > node个数，比较复杂，如下：
					   这里有两个循环，大循环是全node，如果全部node依次下来，还有IRQ没有分配完毕，就会再运行一次全node，确保IRQ全部被分配。
					   小循环是node级别的
						a. 先用 未申请的亲和IRQ个数/node个数得出在一个node中的亲和性IRQ个数，例如 9/4 = 2,那么，放在第一个node里面的就是2个IRQ，剩余7个IRQ,会在下一个node中，用7/3=2,第三次是 5/2=2,第四次是3/1=3。4个node依次是: 2,2,2,3的IRQ分布。
						b. 获取node中活动CPU个数，通过 min（活动CPU个数，预期亲和IRQ个数)，根据这个node的活动CPU个数，得到可支持的亲和IRQ个数
						c. 根据活动CPU个数，配合 申请亲和性IRQ个数，可以知道 每个IRQ绑定多少个CPU，以及是否有CPU剩余（ extra_vecs ）
						d. 用for循环开始申请亲和性IRQ，如果有CPU剩余，那么给当前IRQ多绑定一个CPU，同时extra_vecs--
					   每一个node无法消化的IRQ，就会累积到下一个node争取消化，一旦按照上面的算法，所有node都无法消化完的IRQ，则推出小循环，在运行一遍全node，把剩余的IRQ消化掉。（由于亲和性IRQ个数 <= CPU，两次全node肯定可以消化完毕）
		5.最后，将 中断申请总数中 尾部的 非亲和IRQ的 mask 设置为 irq_default_affinity
	2. 为entry申请空间，将亲和性mask拷贝到entry的元素中，并填充一些其他配置信息，比如IRQ个数等。
   		 -》alloc_msi_entry
	2. 调用 pci_msi_setup_msi_irqs 
     -》 pci_msi_setup_msi_irqs
	我们走的路径是msi_domain_alloc_irqs
	-> msi_domain_alloc_irqs
	
	    -》msi_domain_prepare_irqs
		这个函数会调用 ITS的 its_msi_prepare 用来为 这个PCIE设备创建对应的ITT。
		-》its_msi_prepare
			

		-》 __irq_domain_alloc_irqs
		  ->irq_domain_alloc_irqs_hierarchy
			调用alloc，在ITS中就是 its_irq_domain_alloc
		   -》 its_irq_domain_alloc
		      
			-》 its_alloc_device_irq
