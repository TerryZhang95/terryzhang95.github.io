# OFDM到最新无线物理层学习笔记


## OFDM对比OFDMA
**正交频分复用（OFDM）：**
- 单用户传输
- 将一整段带宽分割成多个子载波，每个子载波相互正交互不干扰
- 发送端将频域信号通过反傅里叶转为时域，透过循环前缀发出
- 接收端去除前缀，时域信号透过傅里叶解析原信号
- 循环前缀（Cyclic Prefix）：保护间隔中插入CP减少由于传输延迟造成的干扰，[参考](https://zhuanlan.zhihu.com/p/164962322)

![ofdm图示](/images/wireless/ofdma/ofdm_compare.png)

**正交频分多址（OFDMA）：**
- 多用户
- 将每一个子信道划分为多个资源单元（resource unit）
- ofdm中多个子载波承担一个包的发送，确定时间内只有一个用户可以发送
- ofdma在同一个时隙内将不同的子载波分给不同用户，结合频域和时域进行多路访问

![ofdm对比ofdma](/images/wireless/ofdma/ofdma_vs_ofdm.png)
