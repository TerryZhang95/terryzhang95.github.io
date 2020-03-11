# Linux内核学习笔记之skb

- [skb 作用](#skb-%e4%bd%9c%e7%94%a8)
- [skb 结构体及操作](#skb-%e7%bb%93%e6%9e%84%e4%bd%93%e5%8f%8a%e6%93%8d%e4%bd%9c)
  - [skb 数据操作](#skb-%e6%95%b0%e6%8d%ae%e6%93%8d%e4%bd%9c)
  - [skb_queue](#skbqueue)
  - [关于skb的size](#%e5%85%b3%e4%ba%8eskb%e7%9a%84size)
- [Reference](#reference)
## skb 作用
## skb 结构体及操作

### skb 数据操作
   - skb_push
     - 在buffer开始之前添加data,添加的长度为len
     - skb->data的指针减去len
     - skb->len增加len
     - 返回skb->data的指针
     ![skb_push](/images/skb_push.png)
   - skb_put
     - 在buffer尾部前添加数据，添加长度为len
     - skb的tail指针获取并没有通过skb->tail获取，而是通过skb_tail_pointer(skb)获取
     - skb->tail指针增加len
     - skb->len增加len
     - 返回skb->tail指针
     ![skb_put](/images/skb_put.png)
   - skb_pull
   - skb_trim
   - skb_copy
   - skb_copy_expand
     - 划重点，如果也是需要复制一个skb，同时又需要增加分配的内存空间时，就使用这个api吧
     ```
     struct sk_buff *skb_copy_expand(const struct sk_buff *skb,
				int newheadroom, int newtailroom,
				gfp_t gfp_mask)
     ```
     - newheadroom 和 newtailroom就是在头和尾想要拓展的空间
     - 返回一个新的skb
### skb_queue
   - Manage packets in a queue structure using struct sk_buff_head
     ```
     struct sk_buff_head {
         struct sk_buff *next;        
         struct sk_buff *prev;        
         __u32 qlen;        
         spinlock_t lock; 
     }; 
     ```
### 关于skb的size
   - 在skb中定义了诸多了个size
   - 先实际测试一下看看各个size都是多少
   ```
   ...
    printk("Test Size_of skb is %d\n",sizeof(skb));
   	printk("And Header is %u, Data is %u, Tail is %u, End is %u\n",skb->head,skb->data, skb->tail, skb->end);
   	printk("len is %d, truesize is %d, datalen is %d, maclen is %d\n",skb->len,skb->truesize,skb->data_len,skb->mac_len);
   	printk("skb api get headroom is %d and tailroom is %d\n",skb_headroom(skb),skb_tailroom(skb));
   	printk("api skb pointer diff is %d\n",skb_end_offset(skb));


     //测试结果
     Mar 10 19:19:06 oem-desktop kernel: [22272.067076] Test Size_of skb is 8
     Mar 10 19:19:06 oem-desktop kernel: [22272.067081] And Header is 3139126016, Data is 3139126080, Tail is 170, End is 192
     Mar 10 19:19:06 oem-desktop kernel: [22272.067084] len is 106, truesize is 768, datalen is 0, maclen is 0
     Mar 10 19:19:06 oem-desktop kernel: [22272.067087] skb api get headroom is 64 and tailroom is 22
     Mar 10 19:19:06 oem-desktop kernel: [22272.067089] api skb pointer diff is 192

   ```
   - 依次分析
     - sizeof是针对数据类型占用内存大小的计算符，因此sizeof(sk_buff)始终为8
     - 从skb->header到skb->data的内存指针差 = headroom的大小 = 64
     - 同理，skb->tail到skb->end内存指针的差值 = tailroom的大小 = 22
     - len是实际数据的长度
     - data_len是非线性的长度，它和maclen后面会具体说
     - headroom+tailroom+len = skb_end_offset(skb) = skb->end
       - 实际上，需要给skb分配的内存 = skb_end_offset(skb) + skb->data_len
     - 其中skb->data到skb->tail中间的内存指针是混乱的，后面继续看
     - truesize是skb的缓冲区大小，但是缓冲区和sk_buff在内存上又是怎样的一层关系呢？
       - skb->truesize=sizeof(struct sk_buff) + SKB_DATA_ALIGN(size)
       - size需要根据x86进行调整，即size = SKB_DATA_ALIGN(size)
  
## Reference
- [Basic functions for sk_buff{}](http://www.skbuff.net/skbbasic.html)
- [How SKB works](http://vger.kernel.org/~davem/skb_data.html)
- [Network Buffers and Memory Management](https://www.linuxjournal.com/article/1312)
- [The Relation between `skb->len` and `skb->data_len` and What They Represent](https://0x657573.wordpress.com/2010/11/22/the-relation-between-skb-len-and-skb-data_len-and-what-they-represent/)
- [skb_source_code](https://github.com/torvalds/linux/blob/master/net/core/skbuff.c)
- [Socket Buffer Functions](https://www.fsl.cs.sunysb.edu/kernel-api/re498.html)
- [skb->truesize，len，datalen，size，等的区别](http://blog.chinaunix.net/uid-26029760-id-1746557.html)
- [内核扩展数据包的方法之-----skb_copy_expand
](https://blog.csdn.net/NW_NW_NW/article/details/73251901)


