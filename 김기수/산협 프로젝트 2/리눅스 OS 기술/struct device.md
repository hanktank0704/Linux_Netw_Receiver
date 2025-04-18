---

```C
						 At the lowest level, every device in a Linux system is represented by an
 * instance of struct device. The device structure contains the information
 * that the device model core needs to model the system. Most subsystems, however, track additional information about the devices they host. As a result, it is rare for devices to be represented by bare device structures; instead, that structure, like kobject structures, is usually embedded with in a higher-level representation of the device.
```

- device member description
    
    * struct device - The basic device structure  
    * @parent: The device's "parent" device, the device to which it is attached.  
    * In most cases, a parent device is some sort of bus or host  
    * controller. If parent is NULL, the device, is a top-level device,  
    * which is not usually what you want.  
    * @p: Holds the private data of the driver core portions of the device.  
    * See the comment of the struct device_private for detail.  
    * @kobj: A top-level, abstract class from which other classes are derived.  
    * @init_name: Initial name of the device.  
    * @type: The type of device.  
    * This identifies the device type and carries type-specific  
    * information.  
    * @mutex: Mutex to synchronize calls to its driver.  
    * @bus: Type of bus device is on.  
    * @driver: Which driver has allocated this  
    * @platform_data: Platform data specific to the device.  
    * Example: For devices on custom boards, as typical of embedded  
    * and SOC based hardware, Linux often uses platform_data to point  
    * to board-specific structures describing devices and how they  
    * are wired. That can include what ports are available, chip  
    * variants, which GPIO pins act in what additional roles, and so  
    * on. This shrinks the "Board Support Packages" (BSPs) and  
    * minimizes board-specific \#ifdefs in drivers.  
    * @driver_data: Private pointer for driver specific info.  
    * @links: Links to suppliers and consumers of this device.  
    * @power: For device power management.  
    * See Documentation/driver-api/pm/devices.rst for details.  
    * @pm_domain: Provide callbacks that are executed during system suspend,  
    * hibernation, system resume and during runtime PM transitions  
    * along with subsystem-level and driver-level callbacks.  
    * @em_pd: device's energy model performance domain  
    * @pins: For device pin management.  
    * See Documentation/driver-api/pin-control.rst for details.  
    * @msi: MSI related data  
    * @numa_node: NUMA node this device is close to.  
    * @dma_ops: DMA mapping operations for this device.  
    * @dma_mask: Dma mask (if dma'ble device).  
    * @coherent_dma_mask: Like dma_mask, but for alloc_coherent mapping as not all  
    * hardware supports 64-bit addresses for consistent allocations  
    * such descriptors.  
    * @bus_dma_limit: Limit of an upstream bridge or bus which imposes a smaller  
    * DMA limit than the device itself supports.  
    * @dma_range_map: map for DMA memory ranges relative to that of RAM  
    * @dma_parms: A low level driver may set these to teach IOMMU code about  
    * segment limitations.  
    * @dma_pools: Dma pools (if dma'ble device).  
    * @dma_mem: Internal for coherent mem override.  
    * @cma_area: Contiguous memory area for dma allocations  
    * @dma_io_tlb_mem: Software IO TLB allocator. Not for driver use.  
    * @dma_io_tlb_pools: List of transient swiotlb memory pools.  
    * @dma_io_tlb_lock: Protects changes to the list of active pools.  
    * @dma_uses_io_tlb: %true if device has used the software IO TLB.  
    * @archdata: For arch-specific additions.  
    * @of_node: Associated device tree node.  
    * @fwnode: Associated device node supplied by platform firmware.  
    * @devt: For creating the sysfs "dev".  
    * @id: device instance  
    * @devres_lock: Spinlock to protect the resource of the device.  
    * @devres_head: The resources list of the device.  
    * @class: The class of the device.  
    * @groups: Optional attribute groups.  
    * @release: Callback to free the device after all references have  
    * gone away. This should be set by the allocator of the  
    * device (i.e. the bus driver that discovered the device).  
    * @iommu_group: IOMMU group the device belongs to.  
    * @iommu: Per device generic IOMMU runtime data  
    * @physical_location: Describes physical location of the device connection  
    * point in the system housing.  
    * @removable: Whether the device can be removed from the system. This  
    * should be set by the subsystem / bus driver that discovered  
    * the device.  
    *  
    * @offline_disabled: If set, the device is permanently online.  
    * @offline: Set after successful invocation of bus type's .offline().  
    * @of_node_reused: Set if the device-tree node is shared with an ancestor  
    * device.  
    * @state_synced: The hardware state of this device has been synced to match  
    * the software state of this device by calling the driver/bus  
    * sync_state() callback.  
    * @can_match: The device has matched with a driver at least once or it is in  
    * a bus (like AMBA) which can't check for matching drivers until  
    * other devices probe successfully.  
    * @dma_coherent: this particular device is dma coherent, even if the  
    * architecture supports non-coherent devices.  
    * @dma_ops_bypass: If set to %true then the dma_ops are bypassed for the  
    * streaming DMA operations (->map_* / ->unmap_* / ->sync_*),  
    * and optionall (if the coherent mask is large enough) also  
    * for dma allocations. This flag is managed by the dma ops  
    * instance from ->dma_supported  
    

```C
/**
 * struct dev_msi_info - Device data related to MSI
 * @domain:	The MSI interrupt domain associated to the device
 * @data:	Pointer to MSI device data
 */
struct dev_msi_info {
\#ifdef CONFIG_GENERIC_MSI_IRQ
	struct irq_domain	*domain;
	struct msi_device_data	*data;
\#endif
};
```