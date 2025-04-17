---
Parameter:
  - sk_buff
Return: int
Location: /net/core/gro.c
---

```c title=skb_gro_receive()
int skb_gro_receive(struct sk_buff *p, struct sk_buff *skb)
{
	struct skb_shared_info *pinfo, *skbinfo = skb_shinfo(skb);
	unsigned int offset = skb_gro_offset(skb);
	unsigned int headlen = skb_headlen(skb);
	unsigned int len = skb_gro_len(skb);
	unsigned int delta_truesize;
	unsigned int gro_max_size;
	unsigned int new_truesize;
	struct sk_buff *lp;
	int segs;

	/* Do not splice page pool based packets w/ non-page pool
	 * packets. This can result in reference count issues as page
	 * pool pages will not decrement the reference count and will
	 * instead be immediately returned to the pool or have frag
	 * count decremented.
	 */
	if (p->pp_recycle != skb->pp_recycle)
		return -ETOOMANYREFS;

	/* pairs with WRITE_ONCE() in netif_set_gro(_ipv4)_max_size() */
	gro_max_size = p->protocol == htons(ETH_P_IPV6) ?
			READ_ONCE(p->dev->gro_max_size) :
			READ_ONCE(p->dev->gro_ipv4_max_size);

	if (unlikely(p->len + len >= gro_max_size || NAPI_GRO_CB(skb)->flush))
		return -E2BIG;

	if (unlikely(p->len + len >= GRO_LEGACY_MAX_SIZE)) {
		if (NAPI_GRO_CB(skb)->proto != IPPROTO_TCP ||
		    (p->protocol == htons(ETH_P_IPV6) &&
		     skb_headroom(p) < sizeof(struct hop_jumbo_hdr)) ||
		    p->encapsulation)
			return -E2BIG;
	}

	segs = NAPI_GRO_CB(skb)->count;
	lp = NAPI_GRO_CB(p)->last;
	pinfo = skb_shinfo(lp);

	if (headlen <= offset) {
		skb_frag_t *frag;
		skb_frag_t *frag2;
		int i = skbinfo->nr_frags;
		int nr_frags = pinfo->nr_frags + i;

		if (nr_frags > MAX_SKB_FRAGS)
			goto merge;

		offset -= headlen;
		pinfo->nr_frags = nr_frags;
		skbinfo->nr_frags = 0;

		frag = pinfo->frags + nr_frags;
		frag2 = skbinfo->frags + i;
		do {
			*--frag = *--frag2;
		} while (--i);

		skb_frag_off_add(frag, offset);
		skb_frag_size_sub(frag, offset);

		/* all fragments truesize : remove (head size + sk_buff) */
		new_truesize = SKB_TRUESIZE(skb_end_offset(skb));
		delta_truesize = skb->truesize - new_truesize;

		skb->truesize = new_truesize;
		skb->len -= skb->data_len;
		skb->data_len = 0;

		NAPI_GRO_CB(skb)->free = NAPI_GRO_FREE;
		goto done;
	} else if (skb->head_frag) {
		int nr_frags = pinfo->nr_frags;
		skb_frag_t *frag = pinfo->frags + nr_frags;
		struct page *page = virt_to_head_page(skb->head);
		unsigned int first_size = headlen - offset;
		unsigned int first_offset;

		if (nr_frags + 1 + skbinfo->nr_frags > MAX_SKB_FRAGS)
			goto merge;

		first_offset = skb->data -
			       (unsigned char *)page_address(page) +
			       offset;

		pinfo->nr_frags = nr_frags + 1 + skbinfo->nr_frags;

		skb_frag_fill_page_desc(frag, page, first_offset, first_size);

		memcpy(frag + 1, skbinfo->frags, sizeof(*frag) * skbinfo->nr_frags);
		/* We dont need to clear skbinfo->nr_frags here */

		new_truesize = SKB_DATA_ALIGN(sizeof(struct sk_buff));
		delta_truesize = skb->truesize - new_truesize;
		skb->truesize = new_truesize;
		NAPI_GRO_CB(skb)->free = NAPI_GRO_FREE_STOLEN_HEAD;
		goto done;
	}

merge:
	/* sk ownership - if any - completely transferred to the aggregated packet */
	skb->destructor = NULL;
	skb->sk = NULL;
	delta_truesize = skb->truesize;
	if (offset > headlen) {
		unsigned int eat = offset - headlen;

		skb_frag_off_add(&skbinfo->frags[0], eat);
		skb_frag_size_sub(&skbinfo->frags[0], eat);
		skb->data_len -= eat;
		skb->len -= eat;
		offset = headlen;
	}

	__skb_pull(skb, offset);

	if (NAPI_GRO_CB(p)->last == p)
		skb_shinfo(p)->frag_list = skb;
	else
		NAPI_GRO_CB(p)->last->next = skb;
	NAPI_GRO_CB(p)->last = skb;
	__skb_header_release(skb);
	lp = p;

done:
	NAPI_GRO_CB(p)->count += segs;
	p->data_len += len;
	p->truesize += delta_truesize;
	p->len += len;
	if (lp != p) {
		lp->data_len += len;
		lp->truesize += delta_truesize;
		lp->len += len;
	}
	NAPI_GRO_CB(skb)->same_flow = 1;
	return 0;
}
```

> 기존의 패킷에다가 skb를 새로 덧붙이게 된다. skb_shared_info에다가 frag를 붙이게 되고, 만약 여유공간이 없다면 새로 만들게 된다.