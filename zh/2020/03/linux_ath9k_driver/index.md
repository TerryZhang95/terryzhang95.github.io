# linux开源驱动mac80211,ath9k开发笔记之发送tx

- [前言](#前言)
- [框架介绍](#框架介绍)
- [ath9k的接口](#ath9k的接口)
  - [ath9k发送的入口-- ath_tx](#ath9k发送的入口---ath_tx)
- [ieee80211的接口](#ieee80211的接口)
  - [ieee80211的入口](#ieee80211的入口)
  - [ieee80211 发送的流程](#ieee80211-发送的流程)
  - [ieee80211 处理的各种header](#ieee80211-处理的各种header)
- [Reference](#reference)

持续更新...
# 前言
本文依据个人理解，也是结合在开发过程中遇到的坑和经验来进行的总结和分析。因为个人着重于ath9k的驱动开发，所以重点将围绕ath9k和mac80211展开。此文档的说明顺序和教科书也会有所差别，先从ath9k开始介绍，然后在追溯到mac80211，想当于由下至上的过程，希望能有所帮助。

# 框架介绍
![ath9k传输和接收](/images/ath9k_path.png)
- 简单来看，发送阶段，userspace（linux内核）向下传的包，也就是所谓的sk_buff，会经过一系列的添加header和tailer的过程，最后进入mac80211
- ath9k，也就是一种网卡的驱动，扮演在mac80211和网卡硬件之间的一个接口（API）
- ath9k会直接调度硬件上包的发送

# ath9k的接口
## ath9k发送的入口-- ath_tx
```
static void ath9k_tx(struct ieee80211_hw *hw,
		     struct ieee80211_tx_control *control,
		     struct sk_buff *skb)
```
- 从mac80211下来的包，会直接call到ath9k_tx这个api
- 在main.c中，定义了
```
struct ieee80211_ops ath9k_ops = {
	.tx 		    = ath9k_tx,
	.start 		    = ath9k_start,
	.stop 		    = ath9k_stop,
    ...
}
```
- 所以后续会以ath9k_ops这个结构体去回调ath9k_tx，那么问题来了，**mac80211准备发包的时候是如何call到ath9k的呢？**
- 回到mac80211中，发送tx的函数是ieee80211_tx()，其中会call到drv_tx()，目测这个是给下层发送的函数
- 在mac80211中，定义drv_tx，这里是进入ath9k的入口
```
static inline void drv_tx(struct ieee80211_local *local,
			  struct ieee80211_tx_control *control,
			  struct sk_buff *skb)
{
	local->ops->tx(&local->hw, control, skb);
}
```
- mac80211中定义ieee80211_local，包含了硬件的信息，其中ops的结构
```
const struct ieee80211_ops *ops; //即为上面定义ath_tx的位置
```
- **那么drv_tx()的skb是哪里来的呢？**
- drv_tx应用在ieee80211_tx_frags中，作用就是把经过mac80211的skb_queue处理并把其中的skb向ath9k发送
- 这个函数应用在__ieee80211_tx中，具体会在下面的ieee80211部分进行说明
```
static bool ieee80211_tx_frags(struct ieee80211_local *local,
			       struct ieee80211_vif *vif,
			       struct ieee80211_sta *sta,
			       struct sk_buff_head *skbs,
			       bool txpending)
```
- **总结**
  - tx的流程：ieee80211_tx() ---> drv_tx() ---> ath9k_tx()

# ieee80211的接口
## ieee80211的入口
```
ieee80211_subif_start_xmit
```
- 先说一下这个函数是如何被call到的
- 在iface的函数中，定义了ieee80211的一系列ops
```
static const struct net_device_ops ieee80211_dataif_ops = {
	.ndo_open		= ieee80211_open,
	.ndo_stop		= ieee80211_stop,
	.ndo_uninit		= ieee80211_uninit,
	.ndo_start_xmit		= ieee80211_subif_start_xmit,
    ...
}
```
- 这个函数的作用：
  - 定义sub-interface数据，也就是要传输的数据，并包含了发送的信息（AP还是sta啊这些）
  - sdata和skb之间的关系？
  - 通过sdata->vif.type区分传输的类型（AP,sta,mesh,adhoc等）
  - 主要看一下AP部分：
    - 第一个工作：将skb->data的ETH_ALEN长度写进hdr.addr1,sdata->vif.addr写进hdr.addr2。
      - skb包含向下发送的信息，sdata则是transmitter本身生成的信息
      - 由MAC header的机理可知，addr1->addr4分别是DA，SA，RA，TA（具体会在mac header的部分说）
      - hdrlen = 24，也就是说mac的header长度是24位。
    - 第二个工作：向skb中放入信息
      - 确定了network的header的指针：skb_network_header(skb);transport layer指针：skb_transport_header(skb)
      - 实际测试，说明transport layer header 信息是在skb->data之后
      ```
      ...
      printk("skb data pointer is %u, and skb transport layer is %u\n",skb->data,skb_transport_header(skb))
      // 结果
      skb data pointer is 3895112744, and skb transport layer is 3895112804
      ```
      - 先后放入了：
        - encaps_data
        - meshhdr,qos(如果有)
        - hdr的内容
      - **那hdr中放的是什么呢**
        - 依然以AP为例
        ```
        if (sdata->vif.type == NL80211_IFTYPE_AP)
    			chanctx_conf = rcu_dereference(sdata->vif.chanctx_conf);
    		if (!chanctx_conf)
    			goto fail_rcu;
    		fc |= cpu_to_le16(IEEE80211_FCTL_FROMDS);
    		/* DA BSSID SA */
    		memcpy(hdr.addr1, skb->data, ETH_ALEN);
    		memcpy(hdr.addr2, sdata->vif.addr, ETH_ALEN);
    		memcpy(hdr.addr3, skb->data + ETH_ALEN, ETH_ALEN);
    		hdrlen = 24;
    		band = chanctx_conf->def.chan->band;
    		break;
        ```
        - 确定了frame control
        - ieee80211_hdr 结构体存入了addr1-addr3
        - 确定了hdrlen是24，24不确定是怎么来的
  - 通过ieee80211_xmit 进行发送
  - 要点：
    - 完成了对skb->data的所有添加
## ieee80211 发送的流程
- ieee80211_subif_start_xmit：上面说过了
- ieee80211_xmit：没有对skb更多的操作，直接call到ieee80211_tx
  - 由于上面对hdr进行了赋值，所以在这里可以
  ```
  	hdr = (struct ieee80211_hdr *) skb->data;
  ```
- ieee80211_tx：
  - ieee80211_tx_prepare：准备tx
  - 将skb放进一个hardware的queue
  ```
  sdata->vif.hw_queue[skb_get_queue_mapping(skb)];
  ```
  - led_len是skb中的数据长度
- __ieee80211_tx: 从skb_queue中取出skb进行处理
  - 同样还是对于sdata->vif.type进行分析
- ieee80211_tx_frags：遍历skb_queue，然后将每个skb进行发送
  - call drv_tx进入ath9k，也就是第一部分说的东西

## ieee80211 处理的各种header
- network header
  - 定义了nh作为network指针到skb->data中间的长度，相关函数如下：
  ```
  static inline unsigned char *skb_network_header(const struct sk_buff *skb)
  {
  	return skb->head + skb->network_header;
  }

  static inline void skb_reset_network_header(struct sk_buff *skb)
  {
  	skb->network_header = skb->data - skb->head;
  }

  static inline void skb_set_network_header(struct sk_buff *skb, const int offset)
  {
  	skb_reset_network_header(skb);
  	skb->network_header += offset;
  }
  ```
  - 回到tx.c
  ```
  nh_pos = skb_network_header(skb) - skb->data;
  ```
    - skb->network_header是skb->head到skb的network header指针之间的距离
    - nh_pos = skb->head + skb->network_header - skb->data  
    所以nh_pos = skb->network_header - headroom(skb)
  - 相同道理可以理解transport header
# Reference 
-[Linux Wi-Fi open source drivers-mac80211, ath9k/ath5k](http://www.campsmur.cat/files/mac80211_intro.pdf)
