```c
static int __must_check tcp_queue_rcv(struct sock *sk, struct sk_buff *skb,
				      bool *fragstolen)
{
	int eaten;
	struct sk_buff *tail = skb_peek_tail(&sk->sk_receive_queue);

	eaten = (tail &&
		 tcp_try_coalesce(sk, tail,
				  skb, fragstolen)) ? 1 : 0;
	tcp_rcv_nxt_update(tcp_sk(sk), TCP_SKB_CB(skb)->end_seq);
	if (!eaten) {
		__skb_queue_tail(&sk->sk_receive_queue, skb);
		skb_set_owner_r(skb, sk);
	}
	return eaten;
}
```

`tcp_try_coalesce()`를 시도한다. 성공할 경우 리턴하고
\_\_skb_queue_tail()로 sk_receive_queue에 skb를 추가한다

tcp_try_coalesce()도 추가적인 조건을 확인하고 skb_try_coalesce()를 실행한다. 

[[tcp_try_coalesce()]]
