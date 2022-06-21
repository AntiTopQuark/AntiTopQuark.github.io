---
layout: post
cid: 47
title: Storm流计算Flume
slug: 47
date: 2021/04/13 17:42:57
updated: 2021/04/13 17:42:57
status: publish
author: AntiTopQuark
categories: 
  - Storm
  - 大数据
tags: 
  - 类型
  - 节点
  - 逻辑
  - 程序
  - 分配
  - 内存
  - 数据库
  - 系统
  - 存储
  - 事务
  - 文件
  - 数据
  - 故障
  - header
  - 模型
  - 过程
  - 原理
  - 架构
  - 服务器
  - 日志
  - null
  - 优先
  - sink
  - source
  - channel
  - flume
  - event
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
# 1.Flume简介
Flume是一种**分布式，可靠且可用**的服务，用于**有效地收集，聚合和传输大量日志数据**，是基于流式的简单灵活的架构。
它具有可靠的**可靠性机制**和许多**故障转移和恢复机制**，具有强大的容错性。
它使用简单的**可扩展数据模型**，允许在线分析应用程序。
# 2.组件
### Source
用于数据的收集。将数据捕获后可以先进行自定义处理，然后将数据封装到事件（event） 里（如event的body），最后将事件推入Channel中。
常见的source:
Avro Source、Exce Source、Spooling Directory Source、NetCat Source、Syslog Source、Syslog TCP Source、Syslog UDP Source、HTTP Source、HDFS Source 等等。

### Channel
用于连接Source和Sink的组件，是数据的缓冲区。它可以将事件暂存到内存中也可以持久化到本地磁盘上， 直到Sink处理完该事件。
常见的channel:
Memory Channel、File Channel、Kafka Channel等等。
### Sink
从channel中取出数据，再将数据存入相应的存储文件系统，数据库，或者提交到远程服务器。
常见的sink:
HDFS sink、Logger sink、Avro sink、File Roll sink、Null sink、HBase sink 等等。
### Interceptor
对Source收集的数据，进行分类或者拦截。可以将多个Interceptor连接形成拦截器链。
分类：自定义分类逻辑，将分类属性(k,v类型)，加入event的headers中，然后使用MultiplexingChannelSelector选择器 选择放入哪个channel中。
拦截：自定义丢弃逻辑，将不要的event设为null即可。
### ChannelSelector
将event放入指定的channel中。
### 2种ChannelSelector:
ReplicatingChannelSelector（默认） ：将事件放入所有channel中。（用于复制，也就是备份）
MultiplexingChannelSelector ：结合Interceptor使用。根据header，将event放到指定的channel中。
sinkgroups中的SinkProcessor
按照指定算法将event分配到sink组的sink中。需要先指定一个sink组，再选择SinkProcessor，它会根据配置的分配方式自动将event分到组里的sink中。注意默认的SinkProcessor中，没有sink组的概念，不需要配置，也就是一对一。
### 2种SinkProcessor:
LoadBalanceSinkProcessor ：负载均衡。可选择分配方式：如随机分配、轮询分配等等
FailoverSinkProcessor ：优先级分配（多用于故障转移）。指定sink的优先级，按优先级分配。

# 3.FLume事务
![59761-u8zgr2gnkun.png](http://www.sukidesu.top/usr/uploads/2020/02/2502725320.png)

# 4.内部原理
![43826-76560n34ut.png](http://www.sukidesu.top/usr/uploads/2020/02/4205946643.png)
# 5.拓扑结构
### 简单串联
![55668-wgz32t14aq.png](http://www.sukidesu.top/usr/uploads/2020/02/4007251295.png)
这种模式将多个flume顺序串联起来，从最初的source到最终的sink传输的存储模式。此模式不建议桥接过多的flume数量，flume数量过多不仅会影响传输速率，而且一旦传输过程中某个节点flume宕机，会影响整个系统。
### 复制和多路复用

![92340-l7a4xgckc6.png](http://www.sukidesu.top/usr/uploads/2020/02/4035555278.png)

### 负载均衡和故障转移
![84230-84x6mgt6nnb.png](http://www.sukidesu.top/usr/uploads/2020/02/714835303.png)

### 聚合
![69665-t18eezq32b.png](http://www.sukidesu.top/usr/uploads/2020/02/4232088655.png)
