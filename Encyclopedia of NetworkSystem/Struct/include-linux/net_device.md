---
Location: /include/linux/netdevice.h
sticker: ""
---

```c title=net_device
struct net_device {
	/* Cacheline organization can be found documented in
	 * Documentation/networking/net_cachelines/net_device.rst.
	 * Please update the document when adding new fields.
	 */

	/* TX read-mostly hotpath */
	__cacheline_group_begin(net_device_read_tx);
	unsigned long long	priv_flags;
	const struct net_device_ops *netdev_ops;
	const struct header_ops *header_ops;
	struct netdev_queue	*_tx;
	netdev_features_t	gso_partial_features;
	unsigned int		real_num_tx_queues;
	unsigned int		gso_max_size;
	unsigned int		gso_ipv4_max_size;
	u16			gso_max_segs;
	s16			num_tc;
	/* Note : dev->mtu is often read without holding a lock.
	 * Writers usually hold RTNL.
	 * It is recommended to use READ_ONCE() to annotate the reads,
	 * and to use WRITE_ONCE() to annotate the writes.
	 */
	unsigned int		mtu;
	unsigned short		needed_headroom;
	struct netdev_tc_txq	tc_to_txq[TC_MAX_QUEUE];
#ifdef CONFIG_XPS
	struct xps_dev_maps __rcu *xps_maps[XPS_MAPS_MAX];
#endif
#ifdef CONFIG_NETFILTER_EGRESS
	struct nf_hook_entries __rcu *nf_hooks_egress;
#endif
#ifdef CONFIG_NET_XGRESS
	struct bpf_mprog_entry __rcu *tcx_egress;
#endif
	__cacheline_group_end(net_device_read_tx);

	/* TXRX read-mostly hotpath */
	__cacheline_group_begin(net_device_read_txrx);
	union {
		struct pcpu_lstats __percpu		*lstats;
		struct pcpu_sw_netstats __percpu	*tstats;
		struct pcpu_dstats __percpu		*dstats;
	};
	unsigned long		state;
	unsigned int		flags;
	unsigned short		hard_header_len;
	netdev_features_t	features;
	struct inet6_dev __rcu	*ip6_ptr;
	__cacheline_group_end(net_device_read_txrx);

	/* RX read-mostly hotpath */
	__cacheline_group_begin(net_device_read_rx);
	struct bpf_prog __rcu	*xdp_prog;
	struct list_head	ptype_specific;
	int			ifindex;
	unsigned int		real_num_rx_queues;
	struct netdev_rx_queue	*_rx;
	unsigned long		gro_flush_timeout;
	int			napi_defer_hard_irqs;
	unsigned int		gro_max_size;
	unsigned int		gro_ipv4_max_size;
	rx_handler_func_t __rcu	*rx_handler;
	void __rcu		*rx_handler_data;
	possible_net_t			nd_net;
#ifdef CONFIG_NETPOLL
	struct netpoll_info __rcu	*npinfo;
#endif
#ifdef CONFIG_NET_XGRESS
	struct bpf_mprog_entry __rcu *tcx_ingress;
#endif
	__cacheline_group_end(net_device_read_rx);

	char			name[IFNAMSIZ];
	struct netdev_name_node	*name_node;
	struct dev_ifalias	__rcu *ifalias;
	/*
	 *	I/O specific fields
	 *	FIXME: Merge these and struct ifmap into one
	 */
	unsigned long		mem_end;
	unsigned long		mem_start;
	unsigned long		base_addr;

	/*
	 *	Some hardware also needs these fields (state,dev_list,
	 *	napi_list,unreg_list,close_list) but they are not
	 *	part of the usual set specified in Space.c.
	 */


	struct list_head	dev_list;
	struct list_head	napi_list;
	struct list_head	unreg_list;
	struct list_head	close_list;
	struct list_head	ptype_all;

	struct {
		struct list_head upper;
		struct list_head lower;
	} adj_list;

	/* Read-mostly cache-line for fast-path access */
	xdp_features_t		xdp_features;
	const struct xdp_metadata_ops *xdp_metadata_ops;
	const struct xsk_tx_metadata_ops *xsk_tx_metadata_ops;
	unsigned short		gflags;

	unsigned short		needed_tailroom;

	netdev_features_t	hw_features;
	netdev_features_t	wanted_features;
	netdev_features_t	vlan_features;
	netdev_features_t	hw_enc_features;
	netdev_features_t	mpls_features;

	unsigned int		min_mtu;
	unsigned int		max_mtu;
	unsigned short		type;
	unsigned char		min_header_len;
	unsigned char		name_assign_type;

	int			group;

	struct net_device_stats	stats; /* not used by modern drivers */

	struct net_device_core_stats __percpu *core_stats;

	/* Stats to monitor link on/off, flapping */
	atomic_t		carrier_up_count;
	atomic_t		carrier_down_count;

#ifdef CONFIG_WIRELESS_EXT
	const struct iw_handler_def *wireless_handlers;
	struct iw_public_data	*wireless_data;
#endif
	const struct ethtool_ops *ethtool_ops;
#ifdef CONFIG_NET_L3_MASTER_DEV
	const struct l3mdev_ops	*l3mdev_ops;
#endif
#if IS_ENABLED(CONFIG_IPV6)
	const struct ndisc_ops *ndisc_ops;
#endif

#ifdef CONFIG_XFRM_OFFLOAD
	const struct xfrmdev_ops *xfrmdev_ops;
#endif

#if IS_ENABLED(CONFIG_TLS_DEVICE)
	const struct tlsdev_ops *tlsdev_ops;
#endif

	unsigned int		operstate;
	unsigned char		link_mode;

	unsigned char		if_port;
	unsigned char		dma;

	/* Interface address info. */
	unsigned char		perm_addr[MAX_ADDR_LEN];
	unsigned char		addr_assign_type;
	unsigned char		addr_len;
	unsigned char		upper_level;
	unsigned char		lower_level;

	unsigned short		neigh_priv_len;
	unsigned short          dev_id;
	unsigned short          dev_port;
	unsigned short		padded;

	spinlock_t		addr_list_lock;
	int			irq;

	struct netdev_hw_addr_list	uc;
	struct netdev_hw_addr_list	mc;
	struct netdev_hw_addr_list	dev_addrs;

#ifdef CONFIG_SYSFS
	struct kset		*queues_kset;
#endif
#ifdef CONFIG_LOCKDEP
	struct list_head	unlink_list;
#endif
	unsigned int		promiscuity;
	unsigned int		allmulti;
	bool			uc_promisc;
#ifdef CONFIG_LOCKDEP
	unsigned char		nested_level;
#endif


	/* Protocol-specific pointers */
	struct in_device __rcu	*ip_ptr;
#if IS_ENABLED(CONFIG_VLAN_8021Q)
	struct vlan_info __rcu	*vlan_info;
#endif
#if IS_ENABLED(CONFIG_NET_DSA)
	struct dsa_port		*dsa_ptr;
#endif
#if IS_ENABLED(CONFIG_TIPC)
	struct tipc_bearer __rcu *tipc_ptr;
#endif
#if IS_ENABLED(CONFIG_ATALK)
	void 			*atalk_ptr;
#endif
#if IS_ENABLED(CONFIG_AX25)
	void			*ax25_ptr;
#endif
#if IS_ENABLED(CONFIG_CFG80211)
	struct wireless_dev	*ieee80211_ptr;
#endif
#if IS_ENABLED(CONFIG_IEEE802154) || IS_ENABLED(CONFIG_6LOWPAN)
	struct wpan_dev		*ieee802154_ptr;
#endif
#if IS_ENABLED(CONFIG_MPLS_ROUTING)
	struct mpls_dev __rcu	*mpls_ptr;
#endif
#if IS_ENABLED(CONFIG_MCTP)
	struct mctp_dev __rcu	*mctp_ptr;
#endif

/*
 * Cache lines mostly used on receive path (including eth_type_trans())
 */
	/* Interface address info used in eth_type_trans() */
	const unsigned char	*dev_addr;

	unsigned int		num_rx_queues;
#define GRO_LEGACY_MAX_SIZE	65536u
/* TCP minimal MSS is 8 (TCP_MIN_GSO_SIZE),
 * and shinfo->gso_segs is a 16bit field.
 */
#define GRO_MAX_SIZE		(8 * 65535u)
	unsigned int		xdp_zc_max_segs;
	struct netdev_queue __rcu *ingress_queue;
#ifdef CONFIG_NETFILTER_INGRESS
	struct nf_hook_entries __rcu *nf_hooks_ingress;
#endif

	unsigned char		broadcast[MAX_ADDR_LEN];
#ifdef CONFIG_RFS_ACCEL
	struct cpu_rmap		*rx_cpu_rmap;
#endif
	struct hlist_node	index_hlist;

/*
 * Cache lines mostly used on transmit path
 */
	unsigned int		num_tx_queues;
	struct Qdisc __rcu	*qdisc;
	unsigned int		tx_queue_len;
	spinlock_t		tx_global_lock;

	struct xdp_dev_bulk_queue __percpu *xdp_bulkq;

#ifdef CONFIG_NET_SCHED
	DECLARE_HASHTABLE	(qdisc_hash, 4);
#endif
	/* These may be needed for future network-power-down code. */
	struct timer_list	watchdog_timer;
	int			watchdog_timeo;

	u32                     proto_down_reason;

	struct list_head	todo_list;

#ifdef CONFIG_PCPU_DEV_REFCNT
	int __percpu		*pcpu_refcnt;
#else
	refcount_t		dev_refcnt;
#endif
	struct ref_tracker_dir	refcnt_tracker;

	struct list_head	link_watch_list;

	u8 reg_state;

	bool dismantle;

	enum {
		RTNL_LINK_INITIALIZED,
		RTNL_LINK_INITIALIZING,
	} rtnl_link_state:16;

	bool needs_free_netdev;
	void (*priv_destructor)(struct net_device *dev);

	/* mid-layer private */
	void				*ml_priv;
	enum netdev_ml_priv_type	ml_priv_type;

	enum netdev_stat_type		pcpu_stat_type:8;

#if IS_ENABLED(CONFIG_GARP)
	struct garp_port __rcu	*garp_port;
#endif
#if IS_ENABLED(CONFIG_MRP)
	struct mrp_port __rcu	*mrp_port;
#endif
#if IS_ENABLED(CONFIG_NET_DROP_MONITOR)
	struct dm_hw_stat_delta __rcu *dm_private;
#endif
	struct device		dev;
	const struct attribute_group *sysfs_groups[4];
	const struct attribute_group *sysfs_rx_queue_group;

	const struct rtnl_link_ops *rtnl_link_ops;

	const struct netdev_stat_ops *stat_ops;

	/* for setting kernel sock attribute on TCP connection setup */
#define GSO_MAX_SEGS		65535u
#define GSO_LEGACY_MAX_SIZE	65536u
/* TCP minimal MSS is 8 (TCP_MIN_GSO_SIZE),
 * and shinfo->gso_segs is a 16bit field.
 */
#define GSO_MAX_SIZE		(8 * GSO_MAX_SEGS)

#define TSO_LEGACY_MAX_SIZE	65536
#define TSO_MAX_SIZE		UINT_MAX
	unsigned int		tso_max_size;
#define TSO_MAX_SEGS		U16_MAX
	u16			tso_max_segs;

#ifdef CONFIG_DCB
	const struct dcbnl_rtnl_ops *dcbnl_ops;
#endif
	u8			prio_tc_map[TC_BITMASK + 1];

#if IS_ENABLED(CONFIG_FCOE)
	unsigned int		fcoe_ddp_xid;
#endif
#if IS_ENABLED(CONFIG_CGROUP_NET_PRIO)
	struct netprio_map __rcu *priomap;
#endif
	struct phy_device	*phydev;
	struct sfp_bus		*sfp_bus;
	struct lock_class_key	*qdisc_tx_busylock;
	bool			proto_down;
	unsigned		wol_enabled:1;
	unsigned		threaded:1;

	struct list_head	net_notifier_list;

#if IS_ENABLED(CONFIG_MACSEC)
	/* MACsec management functions */
	const struct macsec_ops *macsec_ops;
#endif
	const struct udp_tunnel_nic_info	*udp_tunnel_nic_info;
	struct udp_tunnel_nic	*udp_tunnel_nic;

	/* protected by rtnl_lock */
	struct bpf_xdp_entity	xdp_state[__MAX_XDP_MODE];

	u8 dev_addr_shadow[MAX_ADDR_LEN];
	netdevice_tracker	linkwatch_dev_tracker;
	netdevice_tracker	watchdog_dev_tracker;
	netdevice_tracker	dev_registered_tracker;
	struct rtnl_hw_stats64	*offload_xstats_l3;

	struct devlink_port	*devlink_port;

#if IS_ENABLED(CONFIG_DPLL)
	struct dpll_pin	__rcu	*dpll_pin;
#endif
#if IS_ENABLED(CONFIG_PAGE_POOL)
	/** @page_pools: page pools created for this netdevice */
	struct hlist_head	page_pools;
#endif
};
```
