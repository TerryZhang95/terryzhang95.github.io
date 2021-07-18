# 工业无线标准记录：Wireless Time Sensitive Network


## 什么是TSN

Packet service主要有三种类型[<sup>2<sup>](#r1)：
- Best Effort: 即尽可能满足吞吐量的要求，而对于时延和可靠性没有保证。应用如网页应用，视频等。
- Constant bit rate: 固定比特率，通常由时分复用 time-division multiplexing（TDM）来提供，来保证固定的时延，接近于0的抖动。但由于通常传输速率很低，会有一个较大的丢包率。应用如通话服务等。
- TSN: 基于best effort，但又规范网络和应用满足一定的要求。对于应用，发包限制在一定带宽下（最大包长、每次传输最大包数）；对于网络而言，为每一次的传输都事先预留好资源（bandwidth，buffer，scheduling等），从而保证一定的latency和丢包率。
  可以看出TSN是介于Best Effort和CBR之间的网络模型。相比较于BE，TSN可以满足时延要求；相比较于CBR，虽然延迟上限略高一点，但丢包率会相对更低。应用如工业控制应用等。

### TSN特点

TSN应用在best-effort有线网络中，拓扑通常由若干网桥和路由组成[<sup>2<sup>](#r1)。

TSN重要组成：
- 同步
  - IEEE 1588 时间同步：
    - 即为所说的precision time protocol (PTP) 协议，由软件实现，目前linux有诸多开源的工具，平均精度为30微秒，标准差54微秒。
    - 微秒级精度基本是软件方案的理论极限，即便有部分专门real-time操作系统会对此优化，但依然难以达到$sub-\mu s$的水平。
    - 除软件外，也有基于硬件的实现的同步方案，比如FPGA上的同步，或者是intel也有生产过指定的芯片，可以达到$ns$级精度，但是这种方法的成本很高，灵活性低。
- 应用与网络之间的合约，对这个合约的要求有：延迟上限，无堵塞；高可靠性灵活性

解决的问题：
- 网络拥堵造成的延迟和抖动
- 传输过程中的丢包

### TSN时间线

- TSN最早是在802.1Q协议（网桥和桥接网络）的延申
- 802.1AS：基于IEEE 1588 同步
- 802.1Qav,802.11Qbv etc.: 增加了traffic shaping，即是为了让每个包都能平稳的被发出，而不是让数据包在传输过程中“打捆”（类似sequence control一类的机制）。同时，对于时间敏感的包，也定义传输的优先级。
- 802.1Qca, 802.1CB etc.: 在traffic shaping和priori的基础上，增加通信路径选择和容错机制。
- 802.1AX-2020 Link Aggregation:
  - 对于有线的聚合，早期的想法是多个物理网线考虑为一个逻辑链路，需要在收发两端进行单独的配置。
  - 在1997年，IEEE 802.3小组提出了Link Aggregation Control Protocol（LACP）。

## Wireless TSN

工业应用[<sup>3<sup>](#r3)分为三类：
- 非实时：并没有时延要求，比如视频传输。
- 软实时：soft real time, 允许较低延迟但并非绝对的同步无抖动，允许较低比例的包超出deadline。
- 硬实时：hard real time，有上限的延迟和极高的可靠性。近期有些比较牛逼的工作，比如w-sharp[<sup>3<sup>](#r4)已经可以达到$10^{-7}$的丢包率。
- latency的要求，是指一个确定的（deterministic）延迟上限（upper bound）

**常见的无线协议：**
- 802.15.4：Zigbee的协议基础，使用CSMA进行信道检测避免冲突，支持频段在$sub-GHz$和2.4GHz，速度在2.4GHz最高到250kb/s。以Zigbee为例，已经在传感器网络被广泛应用，原生支持mesh网络。蓝牙协议也基于此，最初设计限制在1Mb/s的速率上，但目前达到5.0版本后速率已经得到大幅的提升，并且也可以实现mesh。蓝牙更适用于可穿戴IoT设备的场景下
- 802.15.4e: Time-synchronized Channel Hopping (TSHC): 更高可靠性，更低功耗，MAC机制上趋近于TDMA，相比较于CSMA可以达到可控的时延。WirelessHart等标准即是基于此
- 802.11 WiFi：WiFi无疑是WTSN领域中最重要的角色，因为便宜的价格、更高的速率、更快速的铺设。限制802.11无法广泛应用于工业应用的原因依然是CSMA信道检测。尽管wifi也提供了PCF和HCCA等机制，但并未广泛应用
- 5G Ultrareliable Low-latency communication（URLLC）：
  - Air Interface： 
    - 支持灵活frame结构，时隙缩小到0.125ms（LTE中time to live（TTL）是1ms）
      - [TTL](https://ltehacks.com/viewtopic.php?t=2752)：一个包最大的传输时间
    - minislots：支持short transmissions（一个slot包括14个OFDM symbols，一个minislot包括2，4，7个symbols）
  - PHY layer:
    - Polar codes and LDPC codes：对短包支持更好
    - massive mimo：提高信号质量
    - mmWave：更大的信道带宽和指向性
    - Reference signaling enhancements:：更低的decoding延迟，省略HARQ...
  - 架构优化：基于SDN的工作来修改协议栈

**应用于无线通信的挑战**

前面提到，TSN解决的主要问题：
- 网络拥堵造成的延迟和抖动
- 传输过程中的丢包

然而这些条件在无线中显然更为苛刻：信道状态不可控，受到的干扰更高，多个设备共享信道，造成延迟和丢包（Packet error rate）更高。


## 工业4.0与工业物联网
**IoT**:
针对民用级物联网，强调设备之间的数据交换

**而industrial Iot (IIoT)**：
强调工业设备之间的智能互联

二者目标不同，简单来说，IoT追求吞吐量，服务多样化；IIoT追求低时延，高可靠性

**Industry 4.0，工业4.0**：
已经被视作一场工业革命，第三次工业革命是人类发明计算机的时候。
工业4.0的初衷是把IoT带入到cyber-physical system（cps）中。
CPS即表示物理世界和网络之间的联通。
工业4.0最早由德国团队提出，致力于通过物联网技术提升工业生产的效率和性能，

## AI赋能WTSN：智能生产
对制造业新的要求：Customized Manufacturing （CM：自定义制造）[<sup>3<sup>](#r5)

**CM特征：**
- smart interconnectivity：对智能网络协议的选择，以提高对网络资源的利用
- 动态配置：也是对生产资源的动态调度
- 大量数据分析
- 深度融合：边缘计算+云服务+CPS（传感器，机器等）+各式调度算法的深度融合

**AI驱动CM：将AI与IIoT同其他学科的结合**
- 提高生产效率和生产质量：tricky的角度，即可以将CV等领域的工作，强加进制造业
- 可预测性维护：调度各种传感器状态，推断设备状态并及时报修
- Smart supply chain：用于预测供应链的不确定性和变化

**AI的局限：**
- 工业数据上传服务器产生的延迟
- 工业环境中难以满足算力的要求

**具体的可研究的角度：**
- 边缘计算对IoT资源的复用，结合强化学习multiagent（edge server作为agent）计算和决策
- 生产资源调度
- intelligent sensing：对生产环境检测和预测，重点在于传感器数据的传输，可与WSN相结合
- Smart factory，open issue：动态调整网络资源；end-to-end （E2E）数据传输，现有解决方案基于SDN（SDIN）
  - E2E，重点在无线传输，保障latency和reliability，主要针对MAC layer设计、routing方案
- 生产线优化（与网络优化相距较远，简单记录）
  - 多agent协作，宏观的AI赋能（销售端推荐系统，零售端价格预测等），生产线动态配置，多生产任务的动态schedule
- Prototype platform设计：
  - layer1：CPS层，传感器，机器人，自动控制
  - layer2：无线传输层，网络协议，mac层设计
  - layer3：计算层，云、边缘服务器，网络整体调度

<!-- ## Blockchain赋能WTSN -->

## Reference
<div id="r1"></div>
- [1] Messenger, John L. "Time-sensitive networking: An introduction." IEEE Communications Standards Magazine 2.2 (2018): 29-33.
<div id="r2"></div>
- [2] Romanov, A. M., Gringoli, F., & Sikora, A. (2020). A precise synchronization method for future wireless TSN networks. IEEE Transactions on Industrial Informatics, 17(5), 3682-3692
<div id="r3"></div>
- [3] Cavalcanti, Dave, et al. "Extending accurate time distribution and timeliness capabilities over the air to enable future wireless industrial automation systems." Proceedings of the IEEE 107.6 (2019): 1132-1152.
<div id="r4"></div>
- [4] Seijo, Óscar, Jesús Alberto López-Fernández, and Iñaki Val. "w-SHARP: Implementation of a high-performance wireless time-sensitive network for low latency and ultra-low cycle time industrial applications." IEEE Transactions on Industrial Informatics 17.5 (2020): 3651-3662.
<div id="r5"></div>
- [5] Sisinni, Emiliano, et al. "Industrial internet of things: Challenges, opportunities, and directions." IEEE transactions on industrial informatics 14.11 (2018): 4724-4734.
<div id="r6"></div>
- [6] Wan, Jiafu, et al. "Artificial-intelligence-driven customized manufacturing factory: key technologies, applications, and challenges." Proceedings of the IEEE 109.4 (2020): 377-398.

