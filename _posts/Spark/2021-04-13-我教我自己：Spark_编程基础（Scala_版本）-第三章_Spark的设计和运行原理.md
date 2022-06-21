---
layout: post
cid: 37
title: 我教我自己：Spark 编程基础（Scala 版本）-第三章 Spark的设计和运行原理
slug: 37
date: 2021/04/13 17:30:33
updated: 2021/04/13 17:30:33
status: publish
author: AntiTopQuark
categories: 
  - Spark
  - 大数据
tags: 
  - value
  - 类型
  - rdd
  - 节点
  - 逻辑
  - 通道
  - 接口
  - 程序
  - 方法
  - 分配
  - 语言
  - 内存
  - 数据库
  - 系统
  - 引擎
  - 存储
  - 查询
  - 用户
  - 进程
  - 对象
  - Python
  - 文件
  - spark
  - 分区
  - 任务
  - map
  - 数据
  - 操作
  - 故障
  - join
  - 原文
  - 记录
  - 框架
  - 机器
  - 协议
  - 客户端
  - 模型
  - 流程
  - 启动
  - 过程
  - 原理
  - 架构
  - 索引
  - 概述
  - 日志
  - 表
  - 共享
  - log
  - 区别
  - key
  - 测试
  - 环境
  - 开发
  - 处理
  - 知识
  - 文档
  - 大数据
  - 转换
  - 计算
  - 流水线
  - 机器学习
  - partition
  - 步骤
  - 问题
  - 编程
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
## 3.1 概述
Spark 具有如下几个主要特点：
- **运行速度快**：Spark 使用先进的有向无环图（Directed Acyclic Graph,DAG）执行引擎，以支持循环数据流与内存计算，基于内存的执行速度可比 Hadoop MapReduce 快上百倍，基于磁盘的执行速度也能快十倍； 
- **容易使用**：Spark 支持使用 Scala、Java、Python 和 R 语言进行编程，简洁的 API 设计有助于用户轻松构建并行程序，并且可以通过 Spark Shell 进行交互式编程； 
- **通用性**：Spark 提供了完整而强大的技术栈，包括 SQL 查询、流式计算、机器学习和图算法组件，这些组件可以无缝整合在同一个应用中，足以应对复杂的计算； 
- **运行模式多样**：Spark 可运行于独立的集群模式中，或者运行于 Hadoop 中，也可运行于 Amazon EC2 等云环境中，并且可以访问 HDFS、Cassandra、HBase、Hive 等多种数据源。
![](http://www.sukidesu.top/usr/uploads/2019/12/3802964748.png)
## 3.2 Spark生态系统
![BDAS 架构](http://www.sukidesu.top/usr/uploads/2019/12/2381645983.png)
Spark 的生态系统主要包含了 Spark Core、Spark SQL、Spark Streaming、MLlib 和 GraphX 等组件，各个组件的具体功能如下。
**Spark Core**:Spark Core 包含 Spark 最基础和最核心的功能，如内存计算、任务调度、部署模式、故障恢复、存储管理等，主要面向批数据处理。Spark Core 建立在统一的抽象 RDD 之上，使其可以以基本一致的方式应对不同的大数据处理场景；需要注意的是，Spark Core 通常被简称为 Spark。 
**Spark SQL**:Spark SQL 是用于结构化数据处理的组件，允许开发人员直接处理 RDD，同时也可查询 Hive、HBase 等外部数据源。Spark SQL 的一个重要特点是其能够统一处理关系表和 RDD，使得开发人员不需要自己编写 Spark 应用程序，开发人员可以轻松地使用 SQL 命令进行查询，并进行更复杂的数据分析。 
**Spark Streaming**:Spark Streaming 是一种流计算框架，可以支持高吞吐量、可容错处理的实时流数据处理，其核心思路是将流数据分解成一系列短小的批处理作业，每个短小的批处理作业都可以使用 Spark Core 进行快速处理。Spark Streaming 支持多种数据输入源，如 Kafka、Flume 和 TCP 套接字等。 
**MLlib**（机器学习）:MLlib 提供了常用机器学习算法的实现，包括聚类、分类、回归、协同过滤等，降低了机器学习的门槛，开发人员只要具备一定的理论知识就能进行机器学习方面的工作。
**GraphX**（图计算）:GraphX 是 Spark 中用于图计算的 API，可认为是 Pregel 在 Spark 上的重写及优化，GraphX 性能良好，拥有丰富的功能和运算符，能在海量数据上自如地运行复杂的图算法。 
需要说明的是，无论是 Spark SQL、Spark Streaming、MLlib 还是 GraphX，都可以使用 Spark Core 的 API 处理问题，它们的方法几乎是通用的，处理的数据也可以共享，不同应用之间的数据可以无缝集成。
![Spark应用场景](http://www.sukidesu.top/usr/uploads/2019/12/2208325165.png)

## 3.3 Spark运行架构
### 3.3.1 基本概念
- RDD：是弹性分布式数据集（Resilient Distributed Dataset）的简称，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型； 
- DAG：是有向无环图（Directed Acyclic Graph）的简称，反映 RDD 之间的依赖关系； 
- Executor：是运行在工作节点（Worker Node）上的一个进程，负责运行任务，并为应用程序存储数据； 
- 应用（Application）：是用户编写的 Spark 应用程序； 
- 任务（Task）：是运行在 Executor 上的工作单元； 
- 作业（Job）：一个作业包含多个 RDD 及作用于相应 RDD 上的各种操作； 
- 阶段（Stage）：是作业的基本调度单位，一个作业会分为多组任务，每组任务被称为「阶段」，或者也被称为「任务集」
### 3.3.2 架构设计
Spark 运行架构包括集群资源管理器（Cluster Manager）、运行作业任务的工作节点（Worker Node）、每个应用的任务控制节点（Driver Program，或简称为 Driver）和每个工作节点上负责具体任务的执行进程（Executor）。其中，集群资源管理器可以是 Spark 自带的资源管理器，也可以是 YARN 或 Mesos 等资源管理框架。可以看出，就系统架构而言，Spark 采用「主从架构」，包含一个 Master（即 Driver）和若干个 Worker。
![运行架构](http://www.sukidesu.top/usr/uploads/2019/12/3948661256.png)
Executor的优点：
- 一是利用多线程来执行具体的任务（Hadoop MapReduce 采用的是进程模型），减少任务的启动开销；
- 二是 Executor 中有一个 BlockManager 存储模块，会将内存和磁盘共同作为存储设备（默认使用内存，当内存不够时，会写到磁盘），当需要多轮迭代计算时，可以将中间结果存储到这个存储模块里，下次需要时，就可以直接读取该存储模块里的数据，而不需要读取 HDFS 等文件系统的数据，因而有效减少了 I/O 开销，或者在交互式查询场景下，预先将表缓存到该存储系统上，从而可以提高读写 I/O 性能。
在 Spark 中，一个应用（Application）由一个任务控制节点（Driver）和若干个作业（Job）构成，一个作业由多个阶段（Stage）构成，一个阶段由多个任务（Task）组成。当执行一个应用时，任务控制节点 Driver 会向集群管理器（Cluster Manager）申请资源，启动 Executor，并向 Executor 发送应用程序代码和文件，然后在 Executor 上执行任务，运行结束后，执行结果会返回给任务控制节点 Driver，写到 HDFS 或者其他数据库中。
![Spark中各种概念之间的相互关系](http://www.sukidesu.top/usr/uploads/2019/12/172261849.png)
### 3.3.3 Spark运行基本流程
![Spark运行基本流程](http://www.sukidesu.top/usr/uploads/2019/12/4276173489.png)
 1. 当一个 Spark 应用被提交时，首先需要为这个应用构建起基本的运行环境，即由任务控制节点（Driver）创建一个SparkContext 对象，由 SparkContext 负责和资源管理器（Cluster Manager）的通信以及进行资源的申请、任务的分配和监控等，SparkContext 会向资源管理器注册并申请运行 Executor的资源，SparkContext 可以看成是应用程序连接集群的通道。
 2. 资源管理器为 Executor 分配资源，并启动 Executor 进程，Executor 运行情况将随着「心跳」发送到资源管理器上。
 3. SparkContext 根据 RDD 的依赖关系构建 DAG 图，DAG 图提交给 DAG调度器（DAGScheduler）进行解析，将 DAG图分解成多个「阶段」（每个阶段都是一个任务集），并且计算出各个阶段之间的依赖关系，然后把一个个「任务集」提交给底层的任务调度器（TaskScheduler）进行处理；Executor向 SparkContext 申请任务，任务调度器将任务分发给 Executor 运行，同时，SparkContext将应用程序代码发放给 Executor。
 4. 任务在 Executor 上运行，把执行结果反馈给任务调度器，然后反馈给 DAG 调度器，运行完毕后写入数据并释放所有资源。
总体而言，Spark 运行架构具有以下几个特点。 
（1）每个应用都有自己专属的 Executor 进程，并且该进程在应用运行期间一直驻留。Executor 进程以多线程的方式运行任务，减少了多进程任务频繁的启动开销，使得任务执行变得非常高效和可靠。 
（2）Spark 运行过程与资源管理器无关，只要能够获取 Executor 进程并保持通信即可。 
（3）Executor 上有一个 BlockManager 存储模块，类似于键值存储系统（把内存和磁盘共同作为存储设备），在处理迭代计算任务时，不需要把中间结果写入到 HDFS 等文件系统，而是直接放在这个存储系统上，后续有需要时就可以直接读取；在交互式查询场景下，也可以把表提前缓存到这个存储系统上，提高读写 I/O 性能。 
（4）任务采用了数据本地性和推测执行等优化机制**。数据本地性是尽量将计算移到数据所在的节点上进行，即「计算向数据靠拢」，因为移动计算比移动数据所占的网络资源要少得多。**而且，Spark 采用了延时调度机制，可以在更大的程度上实现执行过程优化。比如，拥有数据的节点当前正被其他的任务占用，那么，在这种情况下是否需要将数据移动到其他的空闲节点呢？答案是不一定。因为，如果经过预测发现当前节点结束当前任务的时间要比移动数据的时间还要少，那么，调度就会等待，直到当前节点可用。
### 3.3.4 RDD的设计与运行原理
#### 1.设计背景
RDD 提供了一个抽象的数据架构，我们不必担心底层数据的分布式特性，只需将具体的应用逻辑表达为一系列转换处理，不同 RDD 之间的转换操作形成依赖关系，可以实现管道化，从而避免了中间结果的存储，大大降低了数据复制、磁盘 I/O 和序列化开销。
#### 2.RDD概念
一个 RDD 就是一个分布式对象集合，本质上是一个只读的分区记录集合，每个 RDD 可以分成多个分区，每个分区就是一个数据集片段，并且一个 RDD 的不同分区可以被保存到集群中不同的节点上，从而可以在集群中的不同节点上进行并行计算。
RDD 提供了一种高度受限的共享内存模型，即 RDD 是只读的记录分区的集合，不能直接修改，只能基于稳定的物理存储中的数据集来创建 RDD，或者通过在其他 RDD 上执行确定的转换操作（如 map、join 和 groupBy）而创建得到新的 RDD。RDD 提供了一组丰富的操作以支持常见的数据运算，分为「行动」（Action）和「转换」（Transformation）两种类型，前者用于执行计算并指定输出的形式，后者指定 RDD 之间的相互依赖关系。
两类操作的主要区别是，转换操作（比如 map、filter、groupBy、join 等）接受 RDD 并返回 RDD，而行动操作（比如 count、collect 等）接受 RDD 但是返回非 RDD（即输出一个值或结果）。RDD 提供的转换接口都非常简单，都是类似 map、filter、groupBy、join 等粗粒度的数据转换操作，而不是针对某个数据项的细粒度修改。
需要说明的是，RDD 采用了惰性调用，即在 RDD 的执行过程中，真正的计算发生在 RDD 的「行动」操作，对于「行动」之前的所有「转换」操作，Spark 只是记录下「转换」操作应用的一些基础数据集以及 RDD 生成的轨迹，即相互之间的依赖关系，而不会触发真正的计算。
![87936-e4i8dnqvicl.png](http://www.sukidesu.top/usr/uploads/2019/12/3812702863.png)
#### 3.RDD的特性
（1）高效的容错性。  在 RDD 的设计中，数据只读，不可修改，如果需要修改数据，必须从父 RDD 转换到子 RDD，由此在不同 RDD 之间建立了血缘关系。所以，RDD 是一种天生具有容错机制的特殊集合，不需要通过数据冗余的方式（比如检查点）实现容错，而只需通过 RDD 父子依赖（血缘）关系重新计算得到丢失的分区来实现容错，无需回滚整个系统，这样就避免了数据复制的高开销，而且重算过程可以在不同节点之间并行进行，实现了高效的容错。此外，RDD 提供的转换操作都是一些粗粒度的操作（比如 map、filter 和 join），RDD 依赖关系只需要记录这种粗粒度的转换操作，而不需要记录具体的数据和各种细粒度操作的日志（比如对哪个数据项进行了修改），这就大大降低了数据密集型应用中的容错开销。 
（2）中间结果持久化到内存。数据在内存中的多个 RDD 操作之间进行传递，不需要「落地」到磁盘上，避免了不必要的读写磁盘开销。 
（3）存放的数据可以是 Java 对象，避免了不必要的对象序列化和反序列化开销。
#### 4.RDD之间的依赖关系
RDD 中不同的操作，会使得不同 RDD 分区之间产生不同的依赖关系。DAG 调度器（DAG Scheduler）根据 RDD 之间的依赖关系，把 DAG 图划分成若干个阶段。RDD 中的依赖关系分为窄依赖（Narrow Dependency）与宽依赖（Wide Dependency），二者的主要区别在于是否包含 Shuffle 操作。
Shuffle 操作 Spark 中的一些操作会触发 Shuffle 过程，这个过程涉及到数据的重新分发，因此，会产生大量的磁盘 I/O 和网络开销。
![12025-yr6pybili0h.png](http://www.sukidesu.top/usr/uploads/2019/12/4119938977.png)

首先，在 Map 端的 Shuffle 写入（Shuffle Write）方面。每一个 Map 任务会根据 Reduce 任务的数量创建出相应的桶（Bucket），因此，桶的数量是 m×r，其中，m 是 Map 任务的个数，r 是 Reduce 任务的个数。Map 任务产生的结果会根据设置的分区（partition）算法填充到每个桶中去。分区算法可以自定义，也可以采用系统默认的算法；默认的算法是根据每个键值对（key,value）的 key，把键值对哈希到不同的桶中去。当 Reduce 任务启动时，它会根据自己任务的 id 和所依赖的 Map 任务的 id，从远端或是本地取得相应的桶，作为 Reduce 任务的输入进行处理。
这里的桶是一个抽象概念，在实现中每个桶可以对应一个文件，也可以对应文件的一部分。但是，从性能角度而言，每个桶对应一个文件的实现方式，会导致 Shuffle 过程生成过多的文件。例如，如果有 1000 个 Map 任务和 1000 个 Reduce 任务，就会生成 100 万个文件，这样会给文件系统带来沉重的负担。

所以，在最新的 Spark 版本中，采用了多个桶写入一个文件的方式。
每个 Map 任务不会为每个 Reduce 任务单独生成一个文件，而是把每个 Map 任务所有的输出数据只写到一个文件中。因为每个 Map 任务中的数据会被分区，所以使用了索引（Index）文件来存储具体 Map 任务输出数据在同一个文件中是如何被分区的信息。Shuffle 过程中每个 Map 任务会产生两个文件，即数据文件和索引文件，其中，数据文件是存储当前 Map 任务的输出结果，而索引文件中则存储了数据文件中的数据的分区信息。下一个阶段的 Reduce 任务就是根据索引文件来获取属于自己处理的那个分区的数据。
在 Reduce 端的 Shuffle 读取（Shuffle Fetch）方面，Spark 假定在大多数应用场景中，Shuffle 数据的排序操作不是必须的，比如在进行词频统计时，如果强制地进行排序，只会使性能变差，因此，Spark 并不在 Reduce 端做归并和排序，而是采用了称为 Aggregator 的机制。**Aggregator 本质上是一个 HashMap，里面的每个元素是KV形式。以词频统计为例，它会将从 Map 端拉取到的每一个（key,value），更新或是插入到 HashMap 中，若在 HashMap 中没有查找到这个 key，则把这个（key,value）插入其中，若查找到这个 key，则把 value 的值累加到 V 上去。这样就不需要预先把所有的（key,value）进行归并和排序，而是来一个处理一个，避免了外部排序这一步骤。**
**但同时需要注意的是，Reduce 任务所拥有的内存，必须足以存放属于自己处理的所有 key 和 value 值，否则就会产生内存溢出问题。**
因此，Spark 文档中建议用户涉及到这类操作的时候尽量增加分区的数量，也就是增加 Map 和 Reduce 任务的数量。增加 Map 和 Reduce 任务的数量虽然可以减小分区的大小，使得内存可以容纳这个分区。但是，在 Shuffle 写入环节，桶的数量是由 Map 和 Reduce 任务的数量决定的，任务越多，桶的数量就越多，就需要更多的缓冲区（Buffer），带来更多的内存消耗。因此，在内存使用方面，我们会陷入一个两难的境地，一方面，为了减少内存的使用，需要采取增加 Map 和 Reduce 任务数量的策略，另一方面，Map 和 Reduce 任务数量的增多，又会带来内存开销更大的问题。最终，为了减少内存的使用，只能将 Aggregator 的操作从内存移到磁盘上进行。也就是说，尽管 Spark 经常被称为「基于内存的分布式计算框架」，但是，它的 Shuffle 过程依然需要把数据写入磁盘。


**窄依赖与宽依赖**
窄依赖表现为一个父 RDD 的分区对应于一个子 RDD 的分区，或多个父 RDD 的分区对应于一个子 RDD 的分区；
比如图 3-12（a）中，RDD1 是 RDD2 的父 RDD，RDD2 是子 RDD，RDD1 的分区 1，对应于 RDD2 的一个分区（即分区 4）；
再比如，RDD6 和 RDD7 都是 RDD8 的父 RDD，RDD6 中的分区（分区 15）和 RDD7 中的分区（分区 18），两者都对应于 RDD8 中的一个分区（分区 21）。 
宽依赖则表现为存在一个父 RDD 的一个分区对应一个子 RDD 的多个分区。
比如图 3-12（b）中，RDD9 是 RDD12 的父 RDD，RDD9 中的分区 24 对应了 RDD12 中的两个分区（即分区 27 和分区 28）。 
![86880-3tt16gkb43z.png](http://www.sukidesu.top/usr/uploads/2019/12/1308886300.png)
总体而言，如果父 RDD 的一个分区只被一个子 RDD 的一个分区所使用就是窄依赖，否则就是宽依赖。窄依赖典型的操作包括 map、filter、union 等，不会包含 Shuffle 操作；宽依赖典型的操作包括 groupByKey、sortByKey 等，通常会包含 Shuffle 操作。对于连接（Join）操作，可以分为两种情况。
（1）对输入进行协同划分，属于窄依赖（见图 3-12（a））。所谓协同划分（Co-partitioned）是指多个父 RDD 的某一分区的所有「键（key）」，落在子 RDD 的同一个分区内，不会产生同一个父 RDD 的某一分区，落在子 RDD 的两个分区的情况。 
（2）对输入做非协同划分，属于宽依赖，如图 3-12（b）所示。

RDD可以通过血缘关系获取足够的信息来重新运算和恢复丢失的数据分区，由此带来了性能的提升。相对而言，在两种依赖关系中，窄依赖的失败恢复更为高效，它只需要根据父 RDD 分区重新计算丢失的分区即可（不需要重新计算所有分区），而且可以并行地在不同节点进行重新计算。而对于宽依赖而言，单个节点失效通常意味着重新计算过程会涉及多个父 RDD 分区，开销较大。此外，Spark 还提供了数据检查点和记录日志，用于持久化中间 RDD，从而使得在进行失败恢复时不需要追溯到最开始的阶段。在进行故障恢复时，Spark 会对数据检查点开销和重新计算 RDD 分区的开销进行比较，从而自动选择最优的恢复策略。
#### 5.阶段划分
Spark 根据 DAG 图中的 RDD 依赖关系，把一个作业分成多个阶段。对于宽依赖和窄依赖而言，窄依赖对于作业的优化很有利。逻辑上，每个 RDD 操作都是一个 fork/join（一种用于并行执行任务的框架），把计算 fork 到每个 RDD 分区，完成计算后对各个分区得到的结果进行 join 操作，然后 fork/join 下一个 RDD 操作。如果把一个 Spark 作业直接翻译到物理实现（即执行完一个 RDD 操作再继续执行另外一个 RDD 操作），是很不经济的。首先，每一个 RDD（即使是中间结果）都需要保存到内存或磁盘中，时间和空间开销大；其次，join 作为全局的路障（Barrier），代价是很昂贵的，所有分区上的计算都要完成以后，才能进行 join 得到结果，这样，作业执行进度就会严重受制于最慢的那个节点。如果子 RDD 的分区到父 RDD 的分区是窄依赖，就可以实施经典的 fusion 优化，把两个 fork/join 合并为一个；如果连续的变换操作序列都是窄依赖，就可以把很多个 fork/join 合并为一个，通过这种合并，不但减少了大量的全局路障（Barrier），而且无需保存很多中间结果 RDD，这样可以极大地提升性能。在 Spark 中，这个合并过程就被称为「流水线（Pipeline）优化」。
Spark 通过分析各个 RDD 之间的依赖关系生成了 DAG，再通过分析各个 RDD 中的分区之间的依赖关系来决定如何划分阶段，具体划分方法是：在 DAG 中进行反向解析，遇到宽依赖就断开（因为宽依赖涉及 Shuffle 操作，无法实现流水线化处理），遇到窄依赖就把当前的 RDD 加入到当前的阶段中（因为窄依赖不会涉及 Shuffle 操作，可以实现流水线化处理）；
6.RDD运行过程
由上述论述可知，把一个 DAG 图划分成多个「阶段」以后，每个阶段都代表了一组关联的、相互之间没有 Shuffle 依赖关系的任务组成的任务集合。每个任务集合会被提交给任务调度器（TaskScheduler）进行处理，由任务调度器将任务分发给 Executor 运行。
![根据 RDD 分区的依赖关系划分阶段](http://www.sukidesu.top/usr/uploads/2019/12/540111487.png)
- （1）创建 RDD 对象； 
- （2）SparkContext 负责计算 RDD 之间的依赖关系，构建 DAG； 
- （3）DAGScheduler 负责把 DAG 图分解成多个阶段，每个阶段中包含了多个任务，每个任务会被任务调度器分发给各个工作节点（Worker Node）上的 Executor 去执行。
![RDD 在 Spark 中的运行过程](http://www.sukidesu.top/usr/uploads/2019/12/3503239781.png)


## 3.4 Spark的部署方式
目前，Spark 支持四种不同类型的部署方式，包括 Local、Standalone、Spark on Mesos 和 Spark on YARN。Local 模式是单机模式，常用于本地开发测试，后三种都属于集群部署模式，用于企业的实际生产环境。
1.Standalone 模式 
与 MapReduce1.0 框架类似，Spark 框架本身也自带了完整的资源调度管理服务，可以独立部署到一个集群中，而不需要依赖其他系统来为其提供资源管理调度服务。当采用 Standalone 模式时，在架构的设计上，Spark 与 MapReduce1.0 完全一致，都是由一个 Master 和若干个 Slave 构成，并且以槽（Slot）作为资源分配单位。不同的是，Spark 中的槽不再像 MapReduce1.0 那样分为 Map 槽和 Reduce 槽，而是只设计了统一的一种槽提供给各种任务来使用。 
2.Spark on Mesos 模式 
Mesos 是一种资源调度管理框架，可以为运行在它上面的 Spark 提供服务。由于 Mesos 和 Spark 存在一定的血缘关系，因此，Spark 这个框架在进行设计开发的时候，就充分考虑到了对 Mesos 的充分支持，因此，相对而言，Spark 运行在 Mesos 上，要比运行在 YARN 上更加灵活、自然。目前，Spark 官方推荐采用这种模式，所以，许多公司在实际应用中也采用该模式。 
3.Spark on YARN 模式 
Spark 可运行于 YARN 之上，与 Hadoop 进行统一部署，即「Spark on YARN」，其架构如图 3-15 所示，资源管理和调度依赖 YARN，分布式存储则依赖 HDFS。
![Spark on YARN 架构](http://www.sukidesu.top/usr/uploads/2019/12/765063301.png)
## 3.5 章总结
深刻理解 Spark 的设计与运行原理，是学习 Spark 的基础。作为一种分布式计算框架，Spark 在设计上充分借鉴吸收了MapReduce 的核心思想，并对 MapReduce 中存在的问题进行了改进，获得了很好的实时性能。 
RDD 是 Spark 的数据抽象，一个 RDD 是一个只读的分布式数据集，可以通过转换操作在转换过程中对 RDD 进行各种变换。一个复杂的 Spark 应用程序，就是通过一次又一次的 RDD 操作组合完成的。RDD 操作包括两种类型，即转换操作和行动操作。Spark 采用了惰性机制，在代码中遇到转换操作时，并不会马上开始计算，而只是记录转换的轨迹，只有当遇到行动操作时，才会触发从头到尾的计算。当遇到行动操作时，就会生成一个作业，这个作业会被划分成若干个阶段，每个阶段包含若干个任务，各个任务会被分发到不同的节点上并行执行。 
Spark 可以采用四种不同的部署方式，包括 Local、Standalone、Spark on Mesos 和 Spark on YARN。Local 模式是单机模式，常用于本地开发测试，后三种都属于集群部署模式，用于企业的实际生产环境。



## 为什么要有惰性执行
优化执行计划,比如
1.两个连续的map操作 其实在源码里面是在记录级别连续执行的 而不是做完一个rdd再去map到下一个rdd (流水线优化)
2.避免进行无意义的计算，在dag图里面选择最短路径


#Spark运行模式
- Local

![65343-usqpijfz77.png](http://www.sukidesu.top/usr/uploads/2020/03/2095886463.png)

- Standalone
![13718-d4lfvyz47t8.png](http://www.sukidesu.top/usr/uploads/2020/03/2402074024.png)

- yarn-client
![37541-gecttqxxw1.png](http://www.sukidesu.top/usr/uploads/2020/03/2638885960.png)

- yarn-cluster
![20251-pnrwhab39lm.png](http://www.sukidesu.top/usr/uploads/2020/03/1205395877.png)

看到这里，开头为什么说yarn-client用于测试环境调试程序；yarn-cluster用于生产环境就清楚了。

1、yarn-client，driver运行在本地客户端，负责调度Application，会与yarn集群产生大量的网络通信，从而导致网卡流量激增。好处是，执行时可以在本地看到所有的log，便于调试。所以一般用于测试环境。

2、yarn-cluster，driver运行在NodeManager，每次运行都是随机分配到NM机器上去，不会有网卡流量激增的问题。缺点就是本地提交后看不到log，只能通过yarn application-logs application id命令来查看。比较麻烦。
————————————————
版权声明：本文为CSDN博主「小鬼喵」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zxr717110454/article/details/80636569
# 广播机制
![17626-vd8rkdnfxra.png](http://www.sukidesu.top/usr/uploads/2020/03/1139586529.png)
![73137-jut12qnu8b8.png](http://www.sukidesu.top/usr/uploads/2020/03/123207712.png)