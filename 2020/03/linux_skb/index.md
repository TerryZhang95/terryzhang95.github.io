# Linux内核学习笔记之skb


## skb 是干什么的
## skb 如何工作
```
struct sk_buff {
/* These two members must be first. */
struct sk_buff		*next;
struct sk_buff		*prev;

struct sk_buff_head	*list;

```
1. headroom,tail
2. reserve
3. 数据操作
   - skb_push
     - 在buffer开始之前添加data
     ![skb_push](/images/skb_push.png)
   - skb_put
     - 在buffer尾部前添加数据】
     ![skb_put](/images/skb_put.png)
   - skb_pull
   - skb_trim
4. skb_queue
   - Manage packets in a queue structure using struct sk_buff_head
     ```
     struct sk_buff_head {
         struct sk_buff *next;        
         struct sk_buff *prev;        
         __u32 qlen;        
         spinlock_t lock; 
     }; 
     ```
5. skb->len 与skb->data_len
   - skb是否为linear？
     - skb->data_len = 0: skb为线性，skb的长度是skb->len
     - skb->data_len != 0: skb不为线性，skb的长度是skb->skb->data_len
     - 
## Reference
- [Basic functions for sk_buff{}](http://www.skbuff.net/skbbasic.html)
- [How SKB works](http://vger.kernel.org/~davem/skb_data.html)
- [Network Buffers and Memory Management](https://www.linuxjournal.com/article/1312)
- [The Relation between `skb->len’ and `skb->data_len’ and What They Represent](https://0x657573.wordpress.com/2010/11/22/the-relation-between-skb-len-and-skb-data_len-and-what-they-represent/)


