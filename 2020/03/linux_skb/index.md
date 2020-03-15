# Linux内核学习笔记之skb

- [skb 作用](#skb-%e4%bd%9c%e7%94%a8)
- [skb 结构体及操作](#skb-%e7%bb%93%e6%9e%84%e4%bd%93%e5%8f%8a%e6%93%8d%e4%bd%9c)
  - [skb 数据操作](#skb-%e6%95%b0%e6%8d%ae%e6%93%8d%e4%bd%9c)
    - [skb的copy操作](#skb%e7%9a%84copy%e6%93%8d%e4%bd%9c)
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
     - 从skb头部向后取data
     - 返回的是指向新data的指针
     ![skb_pull](/images/skb_pull.png)
   - skb_trim
     - 从skb尾部取data
     ```
     void skb_trim(struct sk_buff *skb, unsigned int len)
      {
      	if (skb->len > len)
      		__skb_trim(skb, len);
      }
     ```
     - __skb_trim(skb, len) -> __skb_set_length(skb,len):只能针对线性数据而言
     ```
     static inline void __skb_set_length(struct sk_buff *skb, unsigned int len)
      {
      	if (WARN_ON(skb_is_nonlinear(skb)))
      		return;
      	skb->len = len;
      	skb_set_tail_pointer(skb, len);
        # 作用是 skb->tail += offset;
      }
     ```
     - 最后则是将skb->tail向后加len
     ![skb_trim](/images/skb_trim.png) 

#### skb的copy操作
skb定义了诸多的skb_copy方式
- skb_copy
 - 复制并返回一个新的skb
 - 适合对skb的数据进行修改时使用
 ```
 struct sk_buff *skb_copy(const struct sk_buff *skb, gfp_t gfp_mask)
 ```
- skb_copy_expand
 - 划重点，如果也是需要复制一个skb，同时又需要增加分配的内存空间时，就使用这个api吧
 ```
 struct sk_buff *skb_copy_expand(const struct sk_buff *skb,
int newheadroom, int newtailroom,
gfp_t gfp_mask)
 ```
 - newheadroom 和 newtailroom就是在头和尾想要拓展的空间
 - 返回一个新的skb
 - 增加headroom或tailroom的size后对于data的指针是如何变化的呢，实际测试一下：
   - 尝试拓展一个skb，打印出head，data，tail，end的指针
   ```
    skb_test = skb_copy_expand(skb,skb_headroom(skb), skb_tailroom(skb)+sizeof(*u8),GFP_KERNEL);

   # 测试结果
    Mar 11 15:38:13 oem-desktop kernel: [95496.079915] And Header is 2641395712, Data is 2641395776, Tail is 415, End is 3776
    Mar 11 15:38:13 oem-desktop kernel: [95496.079919] api skb pointer diff is 3776
    Mar 11 15:38:13 oem-desktop kernel: [95496.079927] And Header is 3226583040, Data is 3226583104, Tail is 415, End is 7872
    Mar 11 15:38:13 oem-desktop kernel: [95496.079930] api skb pointer diff is 7872
   ```
   - size增大的数值并不一致，后面继续看
   
- skb_copy_to_linear_data：在skb中复制进新的data
  ```
  memcpy(skb->data, from, len);
  ```
- skb_copy_to_linear_data_offset：在skb->data后的offset的位置，复制上data
  ```
  memcpy(skb->data + offset, from, len);
  ```


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
     
     - len是实际数据的长度，data_len是非线性的长度
       - **什么是线性长度**
       - 如果数据都是存储在skb->head和skb->tail中间，即为线性，skb->data_len = 0
       - 如果非线性，skb->len = skb->len - skb->data_len
       ```
       skb->data_len =   struct skb_shared_info->frags[0...struct skb_shared_info->nr_frags].size + size of data in struct skb_shared_info->frag_list
       ```

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
- [skb_source_header_file](https://github.com/torvalds/linux/blob/master/include/linux/skbuff.h)
- [Socket Buffer Functions](https://www.fsl.cs.sunysb.edu/kernel-api/re498.html)
- [skb->truesize，len，datalen，size，等的区别](http://blog.chinaunix.net/uid-26029760-id-1746557.html)
- [内核扩展数据包的方法之-----skb_copy_expand](https://blog.csdn.net/NW_NW_NW/article/details/73251901)
- [What is SKB in Linux kernel? What are SKB operations? Memory Representation of SKB? How to send packet out using skb operations?](http://amsekharkernel.blogspot.com/2014/08/what-is-skb-in-linux-kernel-what-are.html)


