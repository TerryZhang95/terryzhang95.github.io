# RTT实现WiFi定位


由于在室内GPS无法覆盖，室内定位技术越发受到重视

室内定位（indoor positioning system，IPS）使用的方法：
- 接收信号强度（Received Signal Strength Indicator，RSSI）：传输距离越长，信号越弱，距离与信号强度之间可以求得关系
  $$
  RSSI=RSSI_0-10\cdot n\cdot \log_{10}(\frac{d}{d_0}+\omega)
  $$
  $d_0, RSS_0$为初始距离和强度；
  $d$为距离；
  $\omega$ 为信道噪声
- Fingerprinting 指纹定位：
  - 预先采集室内各个位置的RSSI信息（后续也有采集channel state information，CSI），然后再将实测的RSSI来与训练数据进行比对（类似机器学习过程）
  - 目前最精准，但可拓展性差，几乎没有实用价值
- 使用CSI进行定位：
  - 每一个信道都有一个CSI
  - 每一个CSI包括所有子载波的信息
  - 常用发法：Angle of Arrival，AOA；Time of Flight，ToF；Roundtrip Time of Flight，RToF...

## RTT定位技术
Round-Trip Time，RTT 定位：
- 比较新的（2016）技术，命名为Fine-Timing Measurement（FTM），支持的协议是802.11mc，类似于RToF
- 目前（2021）仅由Intel AC 8260 支持，且貌似不太准备继续发展
- 由芯片和物理层直接在天线发送和接收时打上时间戳
- **无需考虑收发两端的同步的问题**

**技术原理**
![时间计算方法示意图(图片来自网络)](/images/wireless/rtt/802.11mc.png)

**局限性**
- RTT目前只适用于Line of Sight，LOS，即信号传输无遮挡；当信号存在反射折射多径等问题时，测量精度大大下降
- 收发端时钟都存在抖动
- 硬件平台以及wifi网卡固件的不同，会导致处理速度的不一致
- 只能点对点，不能一对多

由于存在这些问题，目前学术上解决方案为，RTT+另外一个测量指标一起来评估

相关的工作：
- RTT与RSSI分别定位，并通过*Kalman Filter*将两者的定位结果相融合，迭代计算定位精度[<sup>1<sup>](#r1)
  - [卡尔曼滤波](https://zhuanlan.zhihu.com/p/39912633)
- 使用MUSIC算法优化RTT计算的结果[<sup>2<sup>](#r2)
  - [MUSIC算法](https://blog.csdn.net/zhangziju/article/details/100730081)，本质属于利用CSI计算AoA定位
  - CSI是数据最为精准的方案，尤其针对反射、多晶问题，但受限于芯片：目前可以采集CSI的芯片有Atheros大部分芯片和Intel 5300
  - 这篇工作CSI和RTT结合的方法：AP配置8260网卡，在旁边放一个5300网卡去采集环境CSI


**实验工具**
- [Enable 802.11mc FTM on Intel 8260](https://github.com/HappyZ/iw_intel8260_localization/wiki/Enable-802.11mc-FTM-on-Intel-8260)

## Reference
<div id="r1"></div>
- [1] Guo, Guangyi, et al. "Indoor smartphone localization: A hybrid WiFi RTT-RSS ranging approach." IEEE Access 7 (2019): 176767-176781.
<div id="r2"></div>
- [2] Jiokeng, Kevin, et al. "When FTM discovered MUSIC: accurate WiFi-based ranging in the presence of multipath." IEEE INFOCOM 2020-IEEE Conference on Computer Communications. IEEE, 2020.
