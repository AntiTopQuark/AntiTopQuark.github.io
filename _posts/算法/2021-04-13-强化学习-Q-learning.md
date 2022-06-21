---
layout: post
cid: 61
title: 强化学习-Q-learning
slug: 61
date: 2021/04/13 17:54:16
updated: 2021/04/13 17:54:16
status: publish
author: AntiTopQuark
categories: 
  - 算法
tags: 
  - 节点
  - 状态
  - 操作
  - 表
  - 公式
  - 房间
  - 表示
  - state
  - 动作
customSummary: 
noThumbInfoStyle: default
outdatedNotice: no
reprint: standard
thumb: 
thumbChoice: default
thumbDesc: 
thumbSmall: 
thumbStyle: default
---



假设有这样的房间
![90157-po0bqtmxsf.png](http://www.sukidesu.top/usr/uploads/2020/03/1583997474.png)
如果将房间表示成点，然后用房间之间的连通关系表示成线，如下图所示：
![16070-g5xw5il7usi.png](http://www.sukidesu.top/usr/uploads/2020/03/514873732.png)
这就是房间对应的图。我们首先将agent（机器人）处于任何一个位置，让他自己走动，直到走到5房间，表示成功。为了能够走出去，我们将每个节点之间设置一定的权重，能够直接到达5的边设置为100，其他不能的设置为0，这样网络的图为：
![05940-3quanaxfvok.png](http://www.sukidesu.top/usr/uploads/2020/03/757567960.png)
Qlearning中，最重要的就是“状态”和“动作”，状态表示处于图中的哪个节点，比如2节点，3节点等等，而动作则表示从一个节点到另一个节点的操作。
首先我们生成一个奖励矩阵矩阵，矩阵中，-1表示不可以通过，0表示可以通过，100表示直接到达终点：
![23912-sd0oltk0i6f.png](http://www.sukidesu.top/usr/uploads/2020/03/1966831261.png)
同时，我们创建一个Q表，表示学习到的经验，与R表同阶，初始化为0矩阵，表示从一个state到另一个state能获得的总的奖励的折现值。
![01058-z6v7l9rsah.png](http://www.sukidesu.top/usr/uploads/2020/03/2309958607.png)
在上面的公式中，S表示当前的状态，a表示当前的动作，s表示下一个状态，a表示下一个动作，λ为贪婪因子，0<λ<1,一般设置为0.8。Q表示的是，在状态s下采取动作a能够获得的期望最大收益，R是立即获得的收益，而未来一期的收益则取决于下一阶段的动作。
所以，Q-learning的学习步骤可以归结为如下：
![29198-w6e5dhvcfvk.png](http://www.sukidesu.top/usr/uploads/2020/03/2487683684.png)
在迭代到收敛之后，我们就可以根据Q-learning来选择我们的路径走出房间。
看一个实际的例子，首先设定λ=0.8，奖励矩阵R和Q矩阵分别初始化为：
![25626-inzxfhusok.png](http://www.sukidesu.top/usr/uploads/2020/03/3986866223.png)
![79007-sewu33o6s8.png](http://www.sukidesu.top/usr/uploads/2020/03/771645286.png)
随机选择一个状态，比如1，查看状态1所对应的R表，也就是1可以到达3或5，随机地，我们选择5，根据转移方程：
![91002-n0aw3c3hawb.png](http://www.sukidesu.top/usr/uploads/2020/03/746430304.png)
于是，Q表为：
![19265-nltqv3pb9qc.png](http://www.sukidesu.top/usr/uploads/2020/03/1780809922.png)
这样，到达目标，一次尝试结束。
接下来再选择一个随机状态，比如3，3对应的下一个状态有（1，2，4都是状态3对应的非负状态），随机地，我们选择1，这样根据算法更新：
![84651-fnvga5l1hgn.png](http://www.sukidesu.top/usr/uploads/2020/03/3200835456.png)
这样，Q表为：
![30590-pg14sfge33o.png](http://www.sukidesu.top/usr/uploads/2020/03/1609991213.png)
经过不停的迭代，最终我们的Q表为：
![38051-qs7w1la3a5p.png](http://www.sukidesu.top/usr/uploads/2020/03/227815604.png)
我们不妨将Q表中的数转移到我们一开始的示意图中：
![03327-j9r49eeai9o.png](http://www.sukidesu.top/usr/uploads/2020/03/2046027660.png)
在得到Q表之后，我们可以根据如下的算法来选择我们的路径：

![80565-x9q3t0b98hr.png](http://www.sukidesu.top/usr/uploads/2020/03/945399350.png)
举例来说，假设我们的初始状态为2，那么根据Q表，我们选择2-3的动作，然后到达状态3之后，我们可以选择1，2，4。但是根据Q表，我们到1可以达到最大的价值，所以选择动作3-1，随后在状态1，我们按价值最大的选择选择动作1-5，所以路径为2-3-1-5.



作者：文哥的学习日记
链接：https://www.jianshu.com/p/29db50000e3f
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
作者：文哥的学习日记
链接：https://www.jianshu.com/p/29db50000e3f
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

作者：文哥的学习日记
链接：https://www.jianshu.com/p/29db50000e3f
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



