# CSI的采集原理


很多文章有写到CSI概念相关的东西，但是并没有从实践层面说明CSI的采集，这篇文章将着重说明如何理解CSI，以及芯片采集的CSI到底是什么样的，该如何去利用。

[TOC]


## 什么是CSI
CSI即是Channel State information，是无线系统中用于描述信道状态的一个**数值**，数学表达式即为
$$ H = e^{-j2 \pi f \tau} $$
其中$f$为子载波（subcarrier）中心频率，$\tau$为天线之间传播时间
（这个公式不理解，可以回去复习信号与系统）

对空间中传输信号的状态，即为
$$Y=HX+N$$
$Y$为接收信号，$X$为发送信号，$N$为噪音

## CSI矩阵的维度
由上面的公式可知，CSI本身只是对于一个传输信号的状态描述值，而不能描述整体信道的状态。
那么对于发送一个信号，**也就是发送一个包**，究竟返回的信道矩阵是什么样呢？

### 子载波
（熟悉这个part直接跳过吧）
在正交频分复用OFDM之前，频分复用FDM被广泛应用，什么是FDM呢？
<img src="/images/CSI/fdm.png" alt="FDM带宽使用" style="zoom: 33%;" />

即是将一个频段，**分割成多个没有overlapping的多个频段**来使用

不过FDM会造成有效带宽的浪费，因为为了避免overlapping，每一个可用频段之间会流出一部分间隔

为了提高带宽的使用，OFDM系统应运而生

<img src="\images\CSI\ofdm" alt="image-20210311002312517" style="zoom:33%;" />

这里不分析OFDM的数学原理，只从本身原理解释

- 从FDM到OFDM，可以理解为让多个频段有overlapping，来实现对带宽的充分利用；即是将FDM多个频段压缩在一起
- 有overlapping如何避免频段之间的interference？这里就用到了多个subcarrier正交的技术
- 如何理解正交？从图像上看，简单来说，**当一个子载波达到波峰时，其余子载波均为0**，达到的效果就是在接收端，根据这个特性做傅里叶变化，即可区分开不同子载波过来的信号，也叫**相干解调**

### CSI的获取

由子载波的特性可知，**当发送一个包，会经由所有子载波进行传输**

而每一个子载波通过一个条传输path，因此即可返回一条CSI信息

如果发送P个包，有$N$个子载波，那么返回的CSI数量为$P\times N$

当然，如何构建CSI矩阵是可以自定义的，比如可以考虑的变量包括

- 一对transceiver包含的天线数$M$，这个在很多文章中也被成为阵元或者sensor；一根天线会对应N个子载波，因此上述情况，返回CSI数量为$M\times P\times N$

- 多径引发的path数量$L$；这个在分析复杂环境信息时常常会用到，表现形式即为

  $$H = \sum_{1}^{L} f(CSI_{single})+N$$

  


## Reference
[OFDM - Orthogonal Frequency Division Multiplexing](https://www.youtube.com/watch?v=KCHO7zlU25Q)
