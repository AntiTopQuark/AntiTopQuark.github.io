---
layout: post
cid: 35
title: Zookeeper简介
slug: 35
date: 2021/04/13 17:15:00
updated: 2021/04/13 17:16:11
status: publish
author: AntiTopQuark
categories: 
  - Hadoop
  - 大数据
tags: 
  - 类型
  - 节点
  - 逻辑
  - 方法
  - 系统
  - 一致性
  - 状态
  - 进程
  - 文件
  - 数据
  - 操作
  - 机器
  - 客户端
  - 模型
  - 流程
  - 过程
  - master
  - 原理
  - 服务器
  - zookeeper
  - leader
  - 选举
  - 创建
  - 锁
  - lock
  - 通知
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


# Zookeeper 入门
## 概述
Zookeeper 是一个开源的分布式的，为分布式应用提供协调服务的 Apache 项目。
![92171-768lywrzvsk.png](http://www.sukidesu.top/usr/uploads/2020/04/543444158.png)
## 特点
![94188-0xf7khtuvx6.png](http://www.sukidesu.top/usr/uploads/2020/04/1395998304.png)
1）Zookeeper：一个领导者（Leader），多个跟随者（Follower）组成的集群。
**2）集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。**
3）全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
4）更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
5）数据更新原子性，一次数据更新要么成功，要么失败。
6）实时性，在一定时间范围内，Client能读到最新数据

## 数据结构
ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树，每个节点称做一 个ZNode。每一个ZNode默认能够存储1MB的数据，每个ZNode都可以通过其路径唯一标识。
![57354-10xiw7qqg6x.png](http://www.sukidesu.top/usr/uploads/2020/04/2459379490.png)

## 应用场景
提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等
ZooKeeper的运用场景(重点)
1.配置文件统一管理
对分布式系统来说,有很多服务部署在不同的服务器上,不同的服务器有自己一套配置,如果需要配置进行调整,则需要对多台服务器逐个的修改,是不便于管理的

在zookeeper中创建/Configuration持久化节点,将配置信息放在其中
每个客户端都监听(Watch机制)该持久节点,如果有新的配置信息,开发者只需要上传到zookeeper这个节点上(更新节点的配置数据),一旦节点内容修改,每个应用都会得到zookeeper的通知,然后从中获得新的配置信息,在系统中完成配置更新;

2.集群管理
所谓集群管理无在乎两点:是否有机器退出和加入,选举master

在某些集群中可能需要其他集群服务器的状态,比如有机器加入或退出集群等,这个时候就可以通过zookeeper进行集群的统一管理

所有机器约定在父目录/GroupMembers下创建临时节点,每一个节点就代表一台机器,然后监听父目录节点的子节点的变化消息
一旦有机器添加和删除,每个机器都会收到通知,接受通知后,每个机器可以在查询节点信息,就可以知道哪个节点添加或者删除,就知道哪个机器进入或退出了,然后进行集群的配置
master选举的话我们一般通过在父节点创建顺序临时节点,选择编号最小的机器即可

3.分布式锁
实现方式:
1.保持独占:单进程
我们把zookeeper上的一个znode看做一把锁,所有的客户端同时创建一个lock临时节点,通过createNode()方法,谁创建成功,谁就获得锁了;业务操作结束后,释放锁删除该节点即可,就算客户端断开临时节点也会删除所以不用担心会形成死锁;
2.控制时序:加锁排队
创建一个父节点,所有的客户端在它下面创建顺序临时节点,我们保证编号最小的获得这个锁就可以了

4.命名服务(服务的发现和注册)
Dubbo使用的就是这个场景,基于ZooKeeper注册中心,命名服务

假设有一个A服务需要调用B服务,但是AB是两个互相独立的服务,A先完成,B还没有完成
A就可以主动获得zookeeper某个节点"/name",A就可以不需要再管了,B一旦完成,B直接去"/name"节点上将地址加入其中,A服务就监听到了节点内容的修改,就能够调用该地址

对于集群的负载均衡在父节点下面添加子节点就可以实现

# 内部原理

## 节点类型
![13422-778zoe48mi6.png](http://www.sukidesu.top/usr/uploads/2020/04/3868040809.png)
（1）持久化目录节点 ：客户端与Zookeeper断开连接后，该节点依旧存在
（2）持久化顺序编号目录节点： 客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号
（3）临时目录节点： 客户端与Zookeeper断开连接后，该节点被删除
（4）临时顺序编号目录节点： 客户端与Zookeeper断开连接后，该节点被删除，只是 Zookeeper给该节点名称进行顺序编号

## 监听器原理
![97893-j2x2zklrcbh.png](http://www.sukidesu.top/usr/uploads/2020/04/2877152670.png)

## 选举机制
1）半数机制：集群中半数以上机器存活，集群可用。所以 Zookeeper 适合安装奇数台服务器。 为什么zookeeper需要设计一个过半数存活机制?为了防止网络脑裂,保证数据的强一致性。因为整个集群中,有可能因为网络问题"脑裂",导致整个集群分为2个甚至多个集群,如果没有过半数存活机制,那么整个zookeeper会变成多个集群,那么zookeeper提供的数据无法再保证数据一致性;
2）Zookeeper 虽然在配置文件中并没有指定 Master 和 Slave。但是，Zookeeper 工作时，是有一个节点为 Leader，其他则为 Follower，Leader 是通过内部的选举机制临时产生的。 
3）以一个简单的例子来说明整个选举的过程。 假设有五台服务器组成的 Zookeeper 集群，它们的 id 从 1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点上，都是一样的。假设这些服务器依序启动，来看看会发生什么，如图所示。 
![25664-s5t1cu3ei09.png](http://www.sukidesu.top/usr/uploads/2020/04/2579481052.png)

（1）服务器 1 启动，发起一次选举。服务器 1 投自己一票。此时服务器 1 票数一票，不够半数以上（3 票），选举无法完成，服务器 1 状态保持为 LOOKING； 
（2）服务器 2 启动，再发起一次选举。服务器 1 和 2 分别投自己一票并交换选票信息：此时服务器 1 发现服务器 2 的 ID 比自己目前投票推举的（服务器 1）大，更改选票为推举服务器 2。此时服务器 1 票数 0 票，服务器 2 票数 2 票，没有半数以上结果，选举无法完成，服务器 1，2 状态保持 LOOKING 
（3）服务器 3 启动，发起一次选举。此时服务器 1 和 2 都会更改选票为服务器 3。此次投票结果：服务器 1 为 0 票，服务器 2 为 0 票，服务器 3 为 3 票。此时服务器 3 的票数已经超过半数，服务器 3 当选 Leader。服务器 1，2 更改状态为 FOLLOWING，服务器 3 更改状态为 LEADING； 
（4）服务器 4 启动，发起一次选举。此时服务器 1，2，3 已经不是 LOOKING 状态，不会更改选票信息。交换选票信息结果：服务器 3 为 3 票，服务器 4 为 1 票。此时服务器 4服从多数，更改选票信息为服务器 3，并更改状态为 FOLLOWING； 
（5）服务器 5 启动，同 4 一样当小弟

详述：
选择机制中的概念
1、Serverid：服务器ID
比如有三台服务器，编号分别是1,2,3。

编号越大在选择算法中的权重越大。

2、Zxid：数据ID
服务器中存放的最大数据ID.

值越大说明数据越新，在选举算法中数据越新权重越大。

3、Epoch：逻辑时钟
或者叫投票的次数，同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加，然后与接收到的其它服务器返回的投票信息中的数值相比，根据不同的值做出不同的判断。

4、Server状态：选举状态
LOOKING，竞选状态。
FOLLOWING，随从状态，同步leader状态，参与投票。
OBSERVING，观察状态,同步leader状态，不参与投票。
LEADING，领导者状态。

选举流程详述
一、首先开始选举阶段，每个Server读取自身的zxid。

二、发送投票信息

   a、首先，每个Server第一轮都会投票给自己。

   b、投票信息包含 ：所选举leader的Serverid，Zxid，Epoch。Epoch会随着选举轮数的增加而递增。

三、接收投票信息

  1、如果服务器B接收到服务器A的数据（服务器A处于选举状态(LOOKING 状态)

     1）首先，判断逻辑时钟值：

　　　　a）如果发送过来的逻辑时钟Epoch大于目前的逻辑时钟。首先，更新本逻辑时钟Epoch，同时清空本轮逻辑时钟收集到的来自其他server的选举数据。然后，判断是否需要更新当前自己的选举leader Serverid。判断规则rules judging：保存的zxid最大值和leader Serverid来进行判断的。先看数据zxid,数据zxid大者胜出;其次再判断leader Serverid,leader Serverid大者胜出；然后再将自身最新的选举结果(也就是上面提到的三种数据（leader Serverid，Zxid，Epoch）广播给其他server)

　　　　b）如果发送过来的逻辑时钟Epoch小于目前的逻辑时钟。说明对方server在一个相对较早的Epoch中，这里只需要将本机的三种数据（leader Serverid，Zxid，Epoch）发送过去就行。

　　　　c）如果发送过来的逻辑时钟Epoch等于目前的逻辑时钟。再根据上述判断规则rules judging来选举leader ，然后再将自身最新的选举结果(也就是上面提到的三种数据（leader  Serverid，Zxid，Epoch）广播给其他server)。

    2）其次，判断服务器是不是已经收集到了所有服务器的选举状态：若是，根据选举结果设置自己的角色(FOLLOWING还是LEADER)，退出选举过程就是了。

最后，若没有收到没有收集到所有服务器的选举状态：也可以判断一下根据以上过程之后最新的选举leader是不是得到了超过半数以上服务器的支持,如果是,那么尝试在200ms内接收一下数据,如果没有新的数据到来,说明大家都已经默认了这个结果,同样也设置角色退出选举过程。

  2、 如果所接收服务器A处在其它状态（FOLLOWING或者LEADING）。

　　　　a)逻辑时钟Epoch等于目前的逻辑时钟，将该数据保存到recvset。此时Server已经处于LEADING状态，说明此时这个server已经投票选出结果。若此时这个接收服务器宣称自己是leader, 那么将判断是不是有半数以上的服务器选举它，如果是则设置选举状态退出选举过程。
　　　　b) 否则这是一条与当前逻辑时钟不符合的消息，那么说明在另一个选举过程中已经有了选举结果，于是将该选举结果加入到outofelection集合中，再根据outofelection来判断是否可以结束选举,如果可以也是保存逻辑时钟，设置选举状态，退出选举过程。


## 写数据流程
![31064-z9hmpw59bdh.png](http://www.sukidesu.top/usr/uploads/2020/04/2843633919.png)

## 同步流程
选完leader以后，zk就进入状态同步过程。

        1. leader等待server连接；

        2 .Follower连接leader，将最大的zxid发送给leader；

        3 .Leader根据follower的zxid确定同步点；

        4 .完成同步后通知follower 已经成为uptodate状态；

        5 .Follower收到uptodate消息后，又可以重新接受client的请求进行服务了。

流程图如下所示：
![06182-ly11s8ezfb.png](http://www.sukidesu.top/usr/uploads/2020/04/255150700.png)

# 其他
## ZooKeeper 的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？ 
（1）部署方式单机模式、集群模式 （2）角色：Leader 和 Follower （3）集群最少需要机器数：3
## ZooKeeper 的常用命令 
![99907-6jtdtvmp6s2.png](http://www.sukidesu.top/usr/uploads/2020/04/2896069567.png)