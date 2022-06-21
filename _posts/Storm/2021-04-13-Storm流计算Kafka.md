---
layout: post
cid: 48
title: Storm流计算Kafka
slug: 48
date: 2021/04/13 17:43:27
updated: 2021/04/13 17:43:27
status: publish
author: AntiTopQuark
categories: 
  - Storm
  - 大数据
tags: 
  - 节点
  - 逻辑
  - 缓冲
  - 接口
  - 实例
  - 分配
  - 系统
  - 一致性
  - 扩展
  - 用户
  - 进程
  - ack
  - 对象
  - 文件
  - 分区
  - 数据
  - 操作
  - 故障
  - 记录
  - 机器
  - 客户端
  - 流程
  - 过程
  - 架构
  - 索引
  - 服务器
  - zookeeper
  - leader
  - 选举
  - set
  - 日志
  - 参数
  - index
  - log
  - key
  - 处理
  - 大数据
  - partition
  - kafka
  - topic
  - 问题
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



<!-- index-menu -->
# 概述
## 定义
Kafka是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域
## 消息队列
![79378-6j0tso5qybm.png](http://www.sukidesu.top/usr/uploads/2020/03/1412205362.png)
**使用消息队列的好处**
- 解耦：允许你独立的扩展或修改两边处理过程，只要确保它们遵守同样的接口约束
- 可恢复性：系统的一部分组件失效时，不会影响整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列的消息仍然可以在系统恢复后被处理
- 缓冲： 有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况
- 灵活性&峰值处理能力：在访问量剧增的情况下，应用仍然需要进行发挥作用，但是这样的突发流量并不多见。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷请求而完全崩溃
- 异步通信
**使用消息队列的两种模式**
1） 点对点模式（一对一，消费者主动拉取数据，消息收到后消息清楚）

![27079-rlfi9i7eiqi.png](http://www.sukidesu.top/usr/uploads/2020/03/2347362921.png)

2）发布/订阅模式（一对多，消费者消费数据之后不会清楚数据）

发布订阅模式有两种形式：消费者主动拉取；消息队列主动发给消费者。
![97209-a2t52stbg7i.png](http://www.sukidesu.top/usr/uploads/2020/03/1888986706.png)

## Kafka基础架构

![19094-ndj99qw35v.png](http://www.sukidesu.top/usr/uploads/2020/03/635511410.png)

1）Producer ：消息生产者，就是向 kafka broker 发消息的客户端； 
2）Consumer ：消息消费者，向 kafka broker 取消息的客户端； 
3）Consumer Group （CG）：消费者组，由多个 consumer 组成。消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。 
4）Broker ：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker
可以容纳多个 topic。 
5）Topic ：可以理解为一个队列，生产者和消费者面向的都是一个 topic； 
6）Partition：为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列； 
7）Replica：副本，为保证集群中的某个节点发生故障时，该节点上的 partition 数据不丢失，且 kafka 仍然能够继续工作，kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本，一个 leader 和若干个 follower。 
8）leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 leader。 
9）follower：每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 follower

## Kafa架构深入

### 工作流程

![21401-vajvd6domb.png](http://www.sukidesu.top/usr/uploads/2020/03/1664182490.png)
Kafka 中消息是以 topic 进行分类的，生产者生产消息，消费者消费消息，都是面向 topic的。 
topic 是逻辑上的概念，而 partition 是物理上的概念，每个 partition 对应于一个 log 文件，该 log 文件中存储的就是 producer 生产的数据。Producer 生产的数据会被不断追加到该log 文件末端，且每条数据都有自己的 offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费。
![39187-1viyjbeoem7.png](http://www.sukidesu.top/usr/uploads/2020/04/351256474.png)
由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位效率低下，Kafka 采取了分片和索引机制，将每个 partition 分为多个 segment。每个 segment对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名规则为：topic 名称+分区序号。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first0,first-1,first-2。 
![40810-hh4pptp2isu.png](http://www.sukidesu.top/usr/uploads/2020/04/417269555.png)
index 和 log 文件以当前 segment 的第一条消息的 offset 命名。下图为 index 文件和 log 文件的结构示意图。
![72094-cva6a2zsbre.png](http://www.sukidesu.top/usr/uploads/2020/04/2606453052.png)
“.index”文件存储大量的索引信息，“.log”文件存储大量的数据，索引文件中的元 数据指向对应数据文件中 message 的物理偏移地址。

### 生产者
**1）分区的原因** 
（1）方便在集群中扩展，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic
又可以有多个 Partition 组成，因此整个集群就可以适应任意大小的数据了； 
（2）可以提高并发，因为可以以 Partition 为单位读写了
**2）分区的原则** 
我们需要将 producer 发送的数据封装成一个 ProducerRecord 对象。 
 ![64859-otzilftn1ls.png](http://www.sukidesu.top/usr/uploads/2020/03/3326272615.png)
（1）指明 partition 的情况下，直接将指明的值直接作为 partiton 值； 
（2）没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition数进行取余得到 partition 值； 
（3）既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法

**3) 数据可靠性的保证 ISR**
Leader 维护了一个动态的 in-syncreplica set (ISR)，意为和 leader 保持同步的 follower 集合。当 ISR 中的follower 完成数据的同步之后，leader就会给follower发送ack。如果 follower长时间未向 leader 同步数据 ，则该follower 将被踢出ISR，该时间阈值由replica.lag.time.max.ms 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader

**4）ack 应答机制**
对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等 ISR 中的 follower 全部接收成功。 所以 Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置
acks 参数配置： 
acks： 
0：producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟，broker 一接收到还没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据； 
1：producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower同步成功之前 leader 故障，那么将会丢失数据；
![58155-x4wruh7lzg.png](http://www.sukidesu.top/usr/uploads/2020/03/2496654800.png)

-1（all）：producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才返回 ack。但是如果在 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，那么会造成数据重复。 

![39223-1knomcu9hnlj.png](http://www.sukidesu.top/usr/uploads/2020/03/4237498004.png)

**5）故常处理细节**
![38319-aqj6qqrm789.png](http://www.sukidesu.top/usr/uploads/2020/03/4044208007.png)
LEO：指的是每个副本最大的 offset； 
HW：指的是消费者能见到的最大的 offset，ISR 队列中最小的 LEO。 
（1）follower 故障 
follower 发生故障后会被临时踢出 ISR，待该follower 恢复后，follower 会读取本地磁盘记录的上次的HW，并将log文件高于 HW的部分截取掉，从HW开始向 leader 进行同步。等该follower的LEO大于等于leader 的HW，即 follower 追上leader 之后，就可以重新加入 ISR 了。 
（2）leader 故障 
leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性，其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader同步数据。 
[kakfa Leader选举机制][1]
注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。

**6)Exactly Once 语义** 

将服务器的 ACK 级别设置为-1，可以保证 Producer 到 Server 之间不会丢失数据，即 At Least Once 语义。
相对的，将服务器 ACK 级别设置为 0，可以保证生产者每条消息只会被发送一次，即 At Most Once 语义。 
At Least Once 可以保证数据不丢失，但是不能保证数据不重复；相对的，At Least Once可以保证数据不重复，但是不能保证数据不丢失。

但是，对于一些非常重要的信息，比如说交易数据，下游数据消费者要求数据既不重复也不丢失，即 Exactly Once 语义。
在 0.11 版本以前的 Kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。 
0.11 版本的 Kafka，引入了一项重大特性：幂等性。所谓的幂等性就是指 Producer 不论向 Server 发送多少次重复数据，Server 端都只会持久化一条。幂等性结合 At Least Once 语义，就构成了 Kafka 的 Exactly Once 语义。即： 
At Least Once + 幂等性 = Exactly Once 
要启用幂等性，只需要将 Producer 的参数中 enable.idompotence 设置为 true 即可。
Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的 Producer 在初始化的时候会被分配一个 PID，发往同一 Partition 的消息会附带 Sequence Number。而Broker 端会对<PID, Partition, SeqNumber>做缓存，当具有相同主键的消息提交时，Broker 只会持久化一条。 但是 PID 重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话的 Exactly Once。

 

### 消费者
#### 消费方式
consumer 采用 pull（拉）模式从 broker 中读取数据。 push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。 
pull 模式不足之处是，如果 kafka 没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一点，Kafka 的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。 

#### 分区分配策略 
一个 consumer group 中有多个 consumer，一个 topic 有多个 partition，所以必然会涉及到 partition 的分配问题，即确定那个 partition 由哪个 consumer 来消费。
在 kafka 中，存在着两种分区分配策略。**一种是 RangeAssignor 分配策略(范围分区)，另一种是 RoundRobinAssignor分配策略(轮询分区)。默认采用 Range 范围分区。** Kafka提供了消费者客户端参数 partition.assignment.strategy 用来设置消费者与订阅主题之间的分区分配策略。默认情况下，此参数的值为：org.apache.kafka.clients.consumer.RangeAssignor，即采用RangeAssignor分配策略

#####  RangeAssignor 范围分区
 Range 范围分区策略是对每个 topic 而言的。首先对同一个 topic 里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。假如现在有 10 个分区，3 个消费者，排序后的分区将会是0,1,2,3,4,5,6,7,8,9；消费者排序完之后将会是C1-0,C2-0,C3-0。通过 partitions数/consumer数来决定每个消费者应该消费几个分区。如果除不尽，那么前面几个消费者将会多消费 1 个分区。
例如，10/3 = 3 余 1 ，除不尽，那么 消费者 C1-0 便会多消费 1 个分区，最终分区分配结果如下
消费者|消费的分区
-|-
C1-0|消费 0,1,2,3 分区
C2-0|消费 4,5,6 分区
C3-0|消费 7,8,9 分区(如果有11 个分区的话，C1-0 将消费0,1,2,3 分区，C2-0 将消费4,5,6,7分区   C3-0 将消费 8,9,10 分区)

Range 范围分区的弊端：
如上，只是针对 1 个 topic 而言，C1-0消费者多消费1个分区影响不是很大。如果有 N 多个 topic，那么针对每个 topic，消费者 C1-0 都将多消费 1 个分区，topic越多，C1-0 消费的分区会比其他消费者明显多消费 N 个分区。这就是 Range 范围分区的一个很明显的弊端了。

##### RoundRobinAssignor 轮询分区
RoundRobin 轮询分区策略，是把所有的 partition 和所有的 consumer 都列出来，然后按照 hascode 进行排序，最后通过轮询算法来分配 partition 给到各个消费者。
轮询分区分为如下两种情况：①同一消费组内所有消费者订阅的消息都是相同的  ②同一消费者组内的消费者订阅的消息不相同    
①如果同一消费组内，所有的消费者订阅的消息都是相同的，那么 RoundRobin 策略的分区分配会是均匀的。
例如：同一消费者组中，有 3 个消费者C0、C1和C2，都订阅了 2 个主题 t0  和 t1，并且每个主题都有 3 个分区(p0、p1、p2)，那么所订阅的所以分区可以标识为t0p0、t0p1、t0p2、t1p0、t1p1、t1p2。最终分区分配结果如下：
消费者|消费的分区
-|-
消费者C0|消费 t0p0 、t1p0 分区
消费者C1|消费 t0p1 、t1p1 分区
消费者C2|消费 t0p2 、t1p2 分区

②如果同一消费者组内，所订阅的消息是不相同的，那么在执行分区分配的时候，就不是完全的轮询分配，有可能会导致分区分配的不均匀。如果某个消费者没有订阅消费组内的某个 topic，那么在分配分区的时候，此消费者将不会分配到这个 topic 的任何分区。
例如：同一消费者组中，有3个消费者C0、C1和C2，他们共订阅了 3 个主题：t0、t1 和 t2，这 3 个主题分别有 1、2、3 个分区(即:t0有1个分区(p0)，t1有2个分区(p0、p1)，t2有3个分区(p0、p1、p2))，即整个消费者所订阅的所有分区可以标识为 t0p0、t1p0、t1p1、t2p0、t2p1、t2p2。具体而言，消费者C0订阅的是主题t0，消费者C1订阅的是主题t0和t1，消费者C2订阅的是主题t0、t1和t2，最终分区分配结果如下：

消费者C0|消费 t0p0
-|-
消费者C1|消费 t1p0 分区
消费者C2	|消费 t1p1、t2p0、t2p1、t2p2 分区
RoundRobin轮询分区的弊端：
从如上实例，可以看到RoundRobin策略也并不是时分完美，这样分配其实并不是最优解，因为完全可以将分区 t1p1 分配给消费者 C1。
所以，如果想要使用RoundRobin 轮询分区策略，必须满足如下两个条件：
①每个消费者订阅的主题，必须是相同的
②每个主题的消费者实例都是相同的。(即：上面的第一种情况，才优先使用 RoundRobin 轮询分区策略)

##### 什么时候触发分区分配策略
当出现以下几种情况时，Kafka 会进行一次分区分配操作，即 Kafka 消费者端的 Rebalance 操作
① 同一个 consumer 消费者组 group.id 中，新增了消费者进来，会执行 Rebalance 操作
② 消费者离开当期所属的 consumer group组。比如 主动停机  或者  宕机
③ 分区数量发生变化时(即 topic 的分区数量发生变化时)
④ 消费者主动取消订阅
Kafka 消费端的 Rebalance 机制，规定了一个 Consumer group 下的所有 consumer 如何达成一致来分配订阅 topic 的每一个分区。而具体如何执行分区策略，就是上面提到的 Range 范围分区 和 RoundRobin 轮询分区 两种内置的分区策略。
Kafka 对于分区分配策略这块，也提供了可插拔式的实现方式，除了上面两种分区分配策略外，我们也可以创建满足自己使用的分区分配策略，即：自定义分区策略

####  offset 的维护 
由于 consumer 在消费过程中可能会出现断电宕机等故障，consumer 恢复后，需要从故障前的位置的继续消费，所以 consumer 需要实时记录自己消费到了哪个 offset，以便故障恢复后继续消费。 
Kafka 0.9 版本之前，consumer 默认将 offset 保存在 Zookeeper 中，从 0.9 版本开始，consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为__consumer_offsets。 

# Zookeeper 在 Kafka 中的作用 
Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 leader 选举等工作。 
Controller 的管理工作都是依赖于 Zookeeper 的。 
以下为 partition 的 leader 选举过程： 
![50088-naynsha39bg.png](http://www.sukidesu.top/usr/uploads/2020/04/3786043638.png)


# Kafka写入流程
![21613-63jyf6oikf.png](http://www.sukidesu.top/usr/uploads/2020/04/2511057940.png)
1. producer先从zookeeper的"/brokers/.../state"节点找到该partition的leader
2. producer将消息发送给该leader
3. leader将消息写入本地log
4. followers从leader pull信息，写入本地log后向leader发送ACK
5. leader收到所有ISR中replica的ACK后，增加HW,并向producer发送ACK

# Kafka消费流程 
1.连接ZK集群，从ZK中拿到对应topic的partition信息和partition的Leader的相关信息
2.连接到对应Leader对应的broker
3.consumer将自己保存的offset发送给Leader
4.Leader根据offset等信息定位到segment（索引文件和日志文件）
5.根据索引文件中的内容，定位到日志文件中该偏移量对应的开始位置读取相应长度的数据并返回给consumer





  [1]: https://blog.csdn.net/Poppy_Evan/article/details/86371991
  
  
