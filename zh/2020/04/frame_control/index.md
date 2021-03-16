# Linux中无线协议的MAC header分析


# 协议及linux内核源码的关系
## 源码部分
回顾linux源码的几个重要知识点：
- skb是不同传输层级之间传递的数据包
- skb中存在head，data，tail，end等多个指针将skb分成几块，用来存储数据
- skb->data后会存放ieee80211_hdr信息，代码的表现即为
  ```
  struct ieee80211_hdr *hdr = (struct ieee80211_hdr *) skb->data;
  ```
- 那么这一部分hdr信息即为协议中MAC层的header信息
## IEEE80211 协议
IEEE 80211协议，定义了MAC frame的形式
![80211frame](/images/802.11_frame.png)

- 2 Byte – Frame Control
- 2 Byte – Duration/ID
- 4×6 Byte – Address 1 – 4
- 2 Byte – Sequence Control
- 2 Byte – QoS control
- 4 Byte – HT Control (only for 802.11n frames)

在源码中的表现即为
```
struct ieee80211_hdr {
	__le16 frame_control;
	__le16 duration_id;
	u8 addr1[ETH_ALEN];
	u8 addr2[ETH_ALEN];
	u8 addr3[ETH_ALEN];
	__le16 seq_ctrl;
	u8 addr4[ETH_ALEN];
	int flag;
} __packed __aligned(2);
```

# Frame control 具体分析
![mac-framecontrol-04](../../../static/images/cwap-mac-framecontrol-04.png)
## Type (2 bits)
```
00– Management Frame
01– Control Frame
10– Data Frame
11– Reserved
```
## Subtype (4 bits)
- Subtype 明确定义了每个包的用途
- 具体refer(https://en.wikipedia.org/wiki/802.11_Frame_Types)
- 其中reserved的位置即为后期开发可以使用的位数
- 在linux源码中，ieee80211.h中，定义了一系列header以及确定subtype类型的function
- 
# Reference
- [CWAP – MAC Header : Frame Control](https://mrncciew.com/2014/09/27/cwap-mac-header-frame-control/)
- [802.11 Frame Types](https://en.wikipedia.org/wiki/802.11_Frame_Types)

