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



**实验工具**
- [Enable 802.11mc FTM on Intel 8260](https://github.com/HappyZ/iw_intel8260_localization/wiki/Enable-802.11mc-FTM-on-Intel-8260)


