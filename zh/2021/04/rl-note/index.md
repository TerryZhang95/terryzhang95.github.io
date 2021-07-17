# 强化学习干货1


## Markov Decision Process (MDP) 整理

**理解强化学习针对的工作场景：**
![强化学习用于理解agent如何和环境进行交互](/images/learning/rl1/basic.jpg)

**理解马尔可夫过程：**
![马尔可夫决策链](/images/learning/rl1/MDP.png)
- 每个圈代表一个状态state
- 每一个箭头表示一个行为action
- 箭头的数字表示转移概率Transition probability
- 每执行一个action会获得一个奖励reward，记为$R$
- 强化学习即通过$R$来优化执行action的策略policy，直到找到最优的policy $\pi$

**计算步骤：**
- 马尔可夫链：在$t+1$时的状态只与$t$相关，即与$t$之前的所有历史信息无关
- 衰减系数$\gamma$
- 确定每一种转换方式的收益，
- 有限finite MDP表示：$P_{ss'}^a=(\mathbb{P}[S_{t+1}=s'| S_t=s,A_t=a])$
- 回报$G_t$, 
- 价值函数 Value function：一个状态的价值（好坏），也就是在当前状态的回报return之和，$v(s)$



**参考：**

- 部分图摘自[UCL&DEEPMIND RL公开课](https://www.davidsilver.uk/teaching/)

- [MDPs and Bellman Equations](https://github.com/dennybritz/reinforcement-learning/tree/master/MDP)

- [《强化学习》第二讲 马尔科夫决策过程](https://zhuanlan.zhihu.com/p/28084942)

- [强化学习（二）马尔科夫决策过程(MDP)](https://www.cnblogs.com/pinard/p/9426283.html)
