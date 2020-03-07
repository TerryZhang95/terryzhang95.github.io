# Linux内核学习笔记之skb


## skb 是干什么的？
## skb 工作方式
1. headroom,tail
2. reserve
3. 数据操作
   - skb_push
     - 
   - skb_put
   - skb_pull
   - skb_trim
4. skb_queue
   ```
   struct sk_buff {
   	/* These two members must be first. */
   	struct sk_buff		*next;
   	struct sk_buff		*prev;

   	struct sk_buff_head	*list;
    ...

   ```
   - Manage packets in a queue structure using struct sk_buff_head
     ![skb_queue](../images/skb1.png)
     ```
     struct sk_buff_head {
         struct sk_buff *next;        
         struct sk_buff *prev;        
         __u32 qlen;        
         spinlock_t lock; 
     }; 
   ```

## Reference
- [Basic functions for sk_buff{}](http://www.skbuff.net/skbbasic.html)
- [How SKB works](http://vger.kernel.org/~davem/skb_data.html)


