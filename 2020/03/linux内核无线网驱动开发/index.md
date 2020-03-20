# ath9k开发笔记之发送tx

- [框架介绍](#%e6%a1%86%e6%9e%b6%e4%bb%8b%e7%bb%8d)
- [ath9k的接口](#ath9k%e7%9a%84%e6%8e%a5%e5%8f%a3)
  - [ath9k发送的入口-- ath_tx](#ath9k%e5%8f%91%e9%80%81%e7%9a%84%e5%85%a5%e5%8f%a3---athtx)
# 框架介绍
![ath9k传输和接收](/images/ath9k_path.png)
- 简单来看，发送阶段，userspace向下传的包，也就是所谓的sk_buff，会经过一系列的添加header和tailer的过程，最后进入mac80211
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
- **总结**
  - tx的流程：ieee80211_tx() ---> drv_tx() ---> ath9k_tx()

