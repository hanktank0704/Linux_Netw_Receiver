---
Location: /include/net/sock.h
---
```c
struct sock {
	/*
	* Now struct inet_timewait_sock also uses sock_common, so please just
	* don't add nothing before this first member (__sk_common) --acme
	*/
	struct sock_common __sk_common;
#define sk_node __sk_common.skc_node
#define sk_nulls_node __sk_common.skc_nulls_node
#define sk_refcnt __sk_common.skc_refcnt
#define sk_tx_queue_mapping __sk_common.skc_tx_queue_mapping
#ifdef CONFIG_SOCK_RX_QUEUE_MAPPING
#define sk_rx_queue_mapping __sk_common.skc_rx_queue_mapping
#endif
  
#define sk_dontcopy_begin __sk_common.skc_dontcopy_begin
#define sk_dontcopy_end __sk_common.skc_dontcopy_end
#define sk_hash __sk_common.skc_hash
#define sk_portpair __sk_common.skc_portpair
#define sk_num __sk_common.skc_num
#define sk_dport __sk_common.skc_dport
#define sk_addrpair __sk_common.skc_addrpair
#define sk_daddr __sk_common.skc_daddr
#define sk_rcv_saddr __sk_common.skc_rcv_saddr
#define sk_family __sk_common.skc_family
#define sk_state __sk_common.skc_state
#define sk_reuse __sk_common.skc_reuse
#define sk_reuseport __sk_common.skc_reuseport
#define sk_ipv6only __sk_common.skc_ipv6only
#define sk_net_refcnt __sk_common.skc_net_refcnt
#define sk_bound_dev_if __sk_common.skc_bound_dev_if
#define sk_bind_node __sk_common.skc_bind_node
#define sk_prot __sk_common.skc_prot
#define sk_net __sk_common.skc_net
#define sk_v6_daddr __sk_common.skc_v6_daddr
#define sk_v6_rcv_saddr __sk_common.skc_v6_rcv_saddr
#define sk_cookie __sk_common.skc_cookie
#define sk_incoming_cpu __sk_common.skc_incoming_cpu
#define sk_flags __sk_common.skc_flags
#define sk_rxhash __sk_common.skc_rxhash
	  
	__cacheline_group_begin(sock_write_rx);
	  
	atomic_t sk_drops;
	__s32 sk_peek_off;
	struct sk_buff_head sk_error_queue;
	struct sk_buff_head sk_receive_queue;
	/*
	* The backlog queue is special, it is always used with
	* the per-socket spinlock held and requires low latency
	* access. Therefore we special case it's implementation.
	* Note : rmem_alloc is in this structure to fill a hole
	* on 64bit arches, not because its logically part of
	* backlog.
	*/
	struct {
		atomic_t rmem_alloc;
		int len;
		struct sk_buff *head;
		struct sk_buff *tail;
	} sk_backlog;
#define sk_rmem_alloc sk_backlog.rmem_alloc
  
	__cacheline_group_end(sock_write_rx);
	
	__cacheline_group_begin(sock_read_rx);
	/* early demux fields */
	struct dst_entry __rcu *sk_rx_dst;
	int         sk_rx_dst_ifindex;
	u32         sk_rx_dst_cookie;
	  
#ifdef CONFIG_NET_RX_BUSY_POLL
	unsigned int sk_ll_usec;
	unsigned int sk_napi_id;
	u16 sk_busy_poll_budget;
	u8 sk_prefer_busy_poll;
#endif
	u8 sk_userlocks;
	int sk_rcvbuf;
  
	struct sk_filter __rcu *sk_filter;
	union {
		struct socket_wq __rcu *sk_wq;
		/* private: */
		struct socket_wq *sk_wq_raw;
		/* public: */
	};
  
	void (*sk_data_ready)(struct sock *sk);
	long sk_rcvtimeo;
	int sk_rcvlowat;
	__cacheline_group_end(sock_read_rx);
	  
	__cacheline_group_begin(sock_read_rxtx);
	int sk_err;
	struct socket *sk_socket;
	struct mem_cgroup *sk_memcg;
#ifdef CONFIG_XFRM
	struct xfrm_policy __rcu *sk_policy[2];
#endif
	__cacheline_group_end(sock_read_rxtx);
	  
	__cacheline_group_begin(sock_write_rxtx);
	socket_lock_t sk_lock;
	u32 sk_reserved_mem;
	int sk_forward_alloc;
	u32 sk_tsflags;
	__cacheline_group_end(sock_write_rxtx);
	  
	__cacheline_group_begin(sock_write_tx);
	int sk_write_pending;
	atomic_t sk_omem_alloc;
	int sk_sndbuf;
	  
	int sk_wmem_queued;
	refcount_t sk_wmem_alloc;
	unsigned long sk_tsq_flags;
	union {
		struct sk_buff *sk_send_head;
		struct rb_root tcp_rtx_queue;
	};
	struct sk_buff_head sk_write_queue;
	u32 sk_dst_pending_confirm;
	u32 sk_pacing_status; /* see enum sk_pacing */
	struct page_frag sk_frag;
	struct timer_list sk_timer;
	  
	unsigned long sk_pacing_rate; /* bytes per second */
	atomic_t sk_zckey;
	atomic_t sk_tskey;
	__cacheline_group_end(sock_write_tx);
	  
	__cacheline_group_begin(sock_read_tx);
	unsigned long sk_max_pacing_rate;
	long sk_sndtimeo;
	u32 sk_priority;
	u32 sk_mark;
	struct dst_entry __rcu *sk_dst_cache;
	netdev_features_t sk_route_caps;
#ifdef CONFIG_SOCK_VALIDATE_XMIT
	struct sk_buff* (*sk_validate_xmit_skb)(struct sock *sk,
							struct net_device *dev,
							struct sk_buff *skb);
#endif
u16 sk_gso_type;
u16 sk_gso_max_segs;
unsigned int sk_gso_max_size;
gfp_t sk_allocation;
u32 sk_txhash;
u8 sk_pacing_shift;
bool sk_use_task_frag;
__cacheline_group_end(sock_read_tx);
  
/*
* Because of non atomicity rules, all
* changes are protected by socket lock.
*/
	u8 sk_gso_disabled : 1,
	sk_kern_sock : 1,
	sk_no_check_tx : 1,
	sk_no_check_rx : 1;
	u8 sk_shutdown;
	u16 sk_type;
	u16 sk_protocol;
	unsigned long sk_lingertime;
	struct proto *sk_prot_creator;
	rwlock_t sk_callback_lock;
	int sk_err_soft;
	u32 sk_ack_backlog;
	u32 sk_max_ack_backlog;
	kuid_t sk_uid;
	spinlock_t sk_peer_lock;
	int sk_bind_phc;
	struct pid *sk_peer_pid;
	const struct cred *sk_peer_cred;
	  
	ktime_t sk_stamp;
#if BITS_PER_LONG==32
	seqlock_t sk_stamp_seq;
#endif
	int sk_disconnects;
	  
	u8 sk_txrehash;
	u8 sk_clockid;
	u8 sk_txtime_deadline_mode : 1,
	sk_txtime_report_errors : 1,
	sk_txtime_unused : 6;
	  
	void *sk_user_data;
#ifdef CONFIG_SECURITY
	void *sk_security;
#endif
	struct sock_cgroup_data sk_cgrp_data;
	void (*sk_state_change)(struct sock *sk);
	void (*sk_write_space)(struct sock *sk);
	void (*sk_error_report)(struct sock *sk);
	int (*sk_backlog_rcv)(struct sock *sk,
							struct sk_buff *skb);
	void (*sk_destruct)(struct sock *sk);
	struct sock_reuseport __rcu *sk_reuseport_cb;
#ifdef CONFIG_BPF_SYSCALL
	struct bpf_local_storage __rcu *sk_bpf_storage;
#endif
	struct rcu_head sk_rcu;
	netns_tracker ns_tracker;
};
```