# 设备树简介

# 设备（struct device）
linux所有设备的最底层抽象就是struct device，这是linux所有设备共有属性的集合体。结构体定义位于include/linux/device.h，如下：

``` C
struct device {
	struct kobject kobj;
	struct device		*parent;

	struct device_private	*p;

	const char		*init_name; /* initial name of the device */
	const struct device_type *type;

	struct bus_type	*bus;		/* type of bus device is on */
	struct device_driver *driver;	/* which driver has allocated this
					   device */
	void		*platform_data;	/* Platform specific data, device
					   core doesn't touch it */
	void		*driver_data;	/* Driver data, set and get with
					   dev_set_drvdata/dev_get_drvdata */
#ifdef CONFIG_PROVE_LOCKING
	struct mutex		lockdep_mutex;
#endif
	struct mutex		mutex;	/* mutex to synchronize calls to
					 * its driver.
					 */

	struct dev_links_info	links;
	struct dev_pm_info	power;
	struct dev_pm_domain	*pm_domain;

#ifdef CONFIG_GENERIC_MSI_IRQ_DOMAIN
	struct irq_domain	*msi_domain;
#endif
#ifdef CONFIG_PINCTRL
	struct dev_pin_info	*pins;
#endif
#ifdef CONFIG_GENERIC_MSI_IRQ
	struct list_head	msi_list;
#endif

	const struct dma_map_ops *dma_ops;
	u64		*dma_mask;	/* dma mask (if dma'able device) */
	u64		coherent_dma_mask;/* Like dma_mask, but for
					     alloc_coherent mappings as
					     not all hardware supports
					     64 bit addresses for consistent
					     allocations such descriptors. */
	u64		bus_dma_limit;	/* upstream dma constraint */
	unsigned long	dma_pfn_offset;

	struct device_dma_parameters *dma_parms;

	struct list_head	dma_pools;	/* dma pools (if dma'ble) */

#ifdef CONFIG_DMA_DECLARE_COHERENT
	struct dma_coherent_mem	*dma_mem; /* internal for coherent mem
					     override */
#endif
#ifdef CONFIG_DMA_CMA
	struct cma *cma_area;		/* contiguous memory area for dma
					   allocations */
#endif
	/* arch specific additions */
	struct dev_archdata	archdata;

	struct device_node	*of_node; /* associated device tree node */
	struct fwnode_handle	*fwnode; /* firmware device node */

#ifdef CONFIG_NUMA
	int		numa_node;	/* NUMA node this device is close to */
#endif
	dev_t			devt;	/* dev_t, creates the sysfs "dev" */
	u32			id;	/* device instance */

	spinlock_t		devres_lock;
	struct list_head	devres_head;

	struct class		*class;
	const struct attribute_group **groups;	/* optional groups */

	void	(*release)(struct device *dev);
	struct iommu_group	*iommu_group;
	struct dev_iommu	*iommu;

	bool			offline_disabled:1;
	bool			offline:1;
	bool			of_node_reused:1;
	bool			state_synced:1;
#if defined(CONFIG_ARCH_HAS_SYNC_DMA_FOR_DEVICE) || \
    defined(CONFIG_ARCH_HAS_SYNC_DMA_FOR_CPU) || \
    defined(CONFIG_ARCH_HAS_SYNC_DMA_FOR_CPU_ALL)
	bool			dma_coherent:1;
#endif
};
```

从它的结构体定义可以看出，device是用来描述设备的一个原子级结构体，这个结构体只关注设备本身，以及父设备（parent），但是对于设备集合以及层级结构，该结构体是不涉及的。在linux内核代码中，我们经常看到，device结构体的指针会封装到很多上层设备的结构体中，比如在pcie设备，acpi设备的结构体中被使用，而一些上层的API的调用栈的最底层也经常使用device作为入参。

有了device结构体这个基石，设备树就变得非常灵活，无论是纵向扩展，还是横向扩展，只要以某一个device作为锚点，其他的设备结构体的封装体现了该设备的不同属性。例如一个GPIO控制器，既是一个platform_device，也是一个acpi_device，但他们都封装了同一个device结构体

# 设备节点（struct fwnode_handle）
在linux内核中，设备必须挂载在一些总线上，这就使得设备间具备部分层级结构以及集合的属性。但是device结构体本身并不能在设备之间构建起集合的联系，因此，linux内核中使用了fwnode_handle结构体将device做了一层封装，通过node的方式来对设备集合进行描述。

