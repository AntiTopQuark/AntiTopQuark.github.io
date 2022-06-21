---
layout: post
cid: 42
title: 我教我自己：Spark 编程基础（Scala 版本）- 第七章Spark Streaming
slug: 42
date: 2021/04/13 17:37:08
updated: 2021/04/13 17:37:08
status: publish
author: AntiTopQuark
categories: 
  - Spark
  - 大数据
tags: 
  - rdd
  - 逻辑
  - 程序
  - int
  - 方法
  - 语句
  - 数据库
  - 系统
  - 引擎
  - 存储
  - 用户
  - 状态
  - 进程
  - 对象
  - 文件
  - spark
  - 任务
  - 数据
  - 操作
  - 记录
  - 框架
  - 机器
  - 客户端
  - 流程
  - 过程
  - 原理
  - 架构
  - apache
  - 服务器
  - p
  - 概述
  - 日志
  - 参数
  - 创建
  - 区别
  - false
  - 安装
  - 软件
  - 开发
  - 处理
  - 单词
  - 大数据
  - val
  - streaming
  - 计算
  - 特征
  - kafka
  - 步骤
  - 问题
  - 编程
  - local
  - zeppelin
  - hadoop
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



# 7.1 流计算概述
## 7.1.1 静态数据和流数据
数据总体上可以分为静态数据和流数据。
**1.静态数据**
![28744-z4i0290qyua.png](http://www.sukidesu.top/usr/uploads/2019/12/2608110154.png)
**2.流数据**
从概念上而言，流数据（或数据流）是指在时间分布和数量上无限的一系列动态数据集合体；数据记录是流数据的最小组成单元。流数据具有如下特征。
- 数据快速持续到达，潜在大小也许是无穷无尽的。 
- 数据来源众多，格式复杂。 
- 数据量大，但是不十分关注存储，一旦流数据中的某个元素经过处理，要么被丢弃，要么被归档存储。 
- 注重数据的整体价值，不过分关注个别数据。 
- 数据顺序颠倒，或者不完整，系统无法控制将要处理的新到达的数据元素的顺序。

## 7.1.2 批量计算和实时计算
对静态数据和流数据的处理，对应着两种截然不同的计算模式：批量计算和实时计算
![72163-owa4hblcvvl.png](http://www.sukidesu.top/usr/uploads/2019/12/137384962.png)
流数据必须采用实时计算，实时计算最重要的一个需求是能够实时得到计算结果，一般要求响应时间为秒级。
## 7.1.3 流计算概念
流计算平台实时获取来自不同数据源的海量数据，通过实时分析处理，获得有价值的信息。 总的来说，流计算秉承一个基本理念，即数据的价值随着时间的流逝而降低。因此，当事件出现时就应该立即进行处理，而不是缓存起来进行批量处理。为了及时处理流数据，就需要一个低延迟、可扩展、高可靠的处理引擎。对于一个流计算系统来说，它应达到如下需求：
- 高性能。处理大数据的基本要求，如每秒处理几十万条数据。 
- 海量式。支持 TB 级甚至是 PB 级的数据规模。 
- 实时性。必须保证一个较低的延迟时间，达到秒级别，甚至是毫秒级别。 
- 分布式。支持大数据的基本架构，必须能够平滑扩展。 
- 易用性。能够快速进行开发和部署。 
- 可靠性。能可靠地处理流数据。
## 7.1.4 流计算处理流程
![41666-fg6h9ckcens.png](http://www.sukidesu.top/usr/uploads/2019/12/2177896302.png)
**1.数据实时采集** 
数据实时采集阶段通常采集多个数据源的海量数据，需要保证实时性、低延迟与稳定可靠。以日志数据为例，由于分布式集群的广泛应用，数据分散存储在不同的机器上，因此需要实时汇总来自不同机器上的日志数据。 目前有许多互联网公司发布的开源分布式日志采集系统均可满足每秒数百 MB 的数据采集和传输需求，如 Facebook 的 Scribe、LinkedIn 的 Kafka、阿里巴巴的 TimeTunnel，以及基于 Hadoop 的 Chukwa 和 Flume 等。 数据采集系统的基本架构一般有 3 个部分（见图 7-6）。
![28073-pwr9xlu3xdg.png](http://www.sukidesu.top/usr/uploads/2019/12/419845218.png)
- Agent：主动采集数据，并把数据推送到 Collector 部分。 
- Collector：接收多个 Agent 的数据，并实现有序、可靠、高性能的转发。 
- Store：存储 Collector 转发过来的数据。 但对流计算来说，一般在 Store 部分不进行数据的存储，而是将采集的数据直接发送给流计算平台进行实时计算。
**2.数据实时计算**
数据实时计算阶段对采集的数据进行实时的分析和计算。
![58784-p1403twk5ym.png](http://www.sukidesu.top/usr/uploads/2019/12/3450780295.png)
**3.实时查询服务** 
流计算的第三个阶段是实时查询服务，经由流计算框架得出的结果可供用户进行实时查询、展示或储存。
# 7.2 Spark Streaming
## 7.2.1 Spark Streaming设计
Spark Streaming 可整合多种输入数据源，如 Kafka、Flume、HDFS，甚至是普通的 TCP 套接字。经处理后的数据可存储至文件系统、数据库，或显示在仪表盘里。
![97041-5ilt858cl35.png](http://www.sukidesu.top/usr/uploads/2019/12/1554548404.png)
Spark Streaming 的基本原理是将实时输入数据流以时间片（通常在 0.5～2 秒之间）为单位进行拆分，然后采用 Spark 引擎以类似批处理的方式处理每个时间片数据，执行流程如图 7-9 所示。
![88577-wknlbw06vij.png](http://www.sukidesu.top/usr/uploads/2019/12/1975976387.png)
Spark Streaming 最主要的抽象是离散化数据流（Discretized Stream,DStream），表示连续不断的数据流。在内部实现上，Spark Streaming 的输入数据按照时间片（如 1 秒）分成一段一段，每一段数据转换为 Spark 中的 RDD，并且对 DStream 的操作都最终被转变为对相应的 RDD 的操作。
![85657-qe6390bktzs.png](http://www.sukidesu.top/usr/uploads/2019/12/1726857589.png)
例如，如图 7-10 所示，在进行单词的词频统计时，一个又一个句子会像流水一样源源不断到达，Spark Streaming 会把数据流切分成一段一段，每段形成一个 RDD，即 RDD @ time 1、RDD @ time 2、RDD@ time 3 和 RDD @ time 4 等，每个 RDD 里面都包含了一些句子，这些 RDD 就构成了一个 DStream （名称为 lines）。对这个 DStream 执行 flatMap 操作时，实际上会被转换成针对每个 RDD 的 flatMap 操作，转换得到的每个新的 RDD 中都包含了一些单词，这些新的 RDD（即 RDD @ result 1、RDD @result 2、RDD @ result 3、RDD @ result 4 等）又构成了一个新的 DStream（名称为 words）。整个流式计算可根据业务的需求对这些中间的结果进一步处理，或者存储到外部设备中。
## 7.2.2 Spark Streaming 和 Storm 对比
Spark Streaming 和 Storm 最大的区别在于，Spark Streaming 无法实现毫秒级的流计算，而 Storm 可以实现毫秒级响应。
Spark Streaming 无法实现毫秒级的流计算，是因为其将流数据分解为一系列批处理作业，在这个过程中，会产生多个 Spark 作业，且每一段数据的处理都会经过 Spark DAG 图分解、任务调度等过程，需要一定的开销，因此，无法实现毫秒级响应。Spark Streaming 难以满足对实时性要求非常高（如高频实时交易）的场景，但足以胜任其他流式准实时计算场景。相比之下，Storm 处理的数据单位为元组，只会产生极小的延迟。 
Spark Streaming 构建在 Spark 上，一方面是因为 Spark 的低延迟执行引擎（100ms+）可以用于实时计算，另一方面，相比于 Storm，RDD 数据集更容易做高效的容错处理。此外，Spark Streaming 采用的小批量处理的方式，使得它可以同时兼容批量和实时数据处理的逻辑和算法，因此，方便了一些需要历史数据和实时数据联合分析的特定应用场合。
## 7.2.3 从「Hadoop+Storm」架构转向 Spark 架构
![采用 Hadoop+Storm 部署方式的一个案例](http://www.sukidesu.top/usr/uploads/2019/12/3093236902.png)
采用 Spark 架构具有如下优点： 
- 实现一键式安装和配置、线程级别的任务监控和告警； 
- 降低硬件集群、软件维护、任务监控和应用开发的难度； 
- 便于做成统一的硬件、计算平台资源池。 
需要说明的是，正如前面介绍的那样，Spark Streaming 无法实现毫秒级的流计算，因此，对于需要毫秒级实时响应的企业应用而言，仍然需要采用流计算框架（如 Storm）。
![97860-sfszuxh0gb.png](http://www.sukidesu.top/usr/uploads/2019/12/973618294.png)
# 7.3 DStream操作概述
## 7.3.1 Spark Streaming 工作机制
Spark Streaming 中，会有一个组件 Receiver，作为一个长期运行的任务（Task）运行在一个 Executor 上，每个 Receiver 都会负责一个 DStream 输入流（如从文件中读取数据的文件流、套接字流或者从 Kafka 中读取的一个输入流等）。Receiver 组件接收到数据源发来的数据后，会提交给 Spark Streaming 程序进行处理。处理后的结果，可以交给可视化组件进行可视化展示，也可以写入到 HDFS、HBase 中。
![44660-3n8htk9zog9.png](http://www.sukidesu.top/usr/uploads/2019/12/1715845052.png)
## 7.3.2 编写 Spark Streaming 程序的基本步骤
- 通过创建输入 DInput DsStream（tream）来定义输入源。流计算处理的数据对象是来自输入源的数据，这些输入源会源源不断产生数据，并发送给 Spark Streaming，由 Receiver 组件接收到以后，交给用户自定义的 Spark Streaming 程序进行处理；
- 通过对 DStream 应用转换操作和输出操作来定义流计算。流计算过程通常是由用户自定义实现的，需要调用各种 DStream 操作实现用户处理逻辑； 
- 调用 StreamingContext 对象的 start()方法来开始接收数据和处理流程； 
- 通过调用 StreamingContext 对象的 awaitTermination()方法来等待流计算进程结束，或者可以通过调用 StreamingContext 对象的 stop()方法来手动结束流计算进程。
## 7.3.3 创建Streaming Context对象
在 RDD 编程中需要生成一个 SparkContext 对象，在 Spark SQL 编程中需要生成一个 SparkSession 对象，同理，如果要运行一个 Spark Streaming 程序，就需要首先生成一个 StreamingContext 对象，它是 Spark Streaming 程序的主入口。
可以从一个 SparkConf 对象创建一个 StreamingContext 对象。登录 Linux 系统后，启动 spark-shell。进入 spark-shell 以后，就已经获得了一个默认的 SparkConext 对象，也就是 sc。因此，可以采用如下方式来创建 StreamingContext 对象：

    import org.apache.spark.streaming._
    val ssc = new StreamingContext（sc，Seconds（1））

new StreamingContext（sc, Seconds(1)）的两个参数中，sc 表示 SparkContext 对象，Seconds(1)表示在对 Spark Streaming 的数据流进行分段时，每 1 秒切成一个分段。可以调整分段大小，比如使用 Seconds(5)就表示每 5 秒切成一个分段。但是，该系统无法实现毫秒级别的分段，因此，Spark Streaming 无法实现毫秒级别的流计算。
如果是编写一个独立的 Spark Streaming 程序，而不是在 spark-shell 中运行，则需要在代码文件中通过如下方式创建 StreamingContext 对象：
```
    import org.apache.spark._ 
    import org.apache.spark.streaming._ 
    val conf = new SparkConf().setAppName（“ TestDStream”）.setMaster（“ local [2]”） 
    val ssc = new StreamingContext（conf，Seconds（1））
```
# 7.4 基本输入源
## 7.4.1 文件流
在文件流的应用场景中，需要编写 Spark Streaming 程序，一直对文件系统中的某个目录进行监听，一旦发现有新的文件生成，Spark Streaming 就会自动把文件内容读取过来，使用用户自定义的处理逻辑进行处理。
**1.在 spark-shell 中创建文件流**
首先，在 Linux 系统中打开第一个终端（为了便于区分多个终端，这里称为「数据源终端」），创建一个 logfile 目录。然后，在 Linux 系统中打开第二个终端（为了便于区分多个终端，这里称为「流计算终端」），启动进入 spark-shell，然后，依次输入如下语句：
```scala
    import org.apache.spark.streaming._
    val ssc = new StreamingContext(sc,Seconds(1))
    val lines = ssc.textFileStream("file:///home/streaming/logfile")
    val words = lines.flatMap(_.split(" "))
    val wordCounts = words.map(x=>(x,1)).reduceByKey(_+_)
    wordCounts.print()
    ssc.start()
    stopByMarkKey(ssc)
    def stopByMarkKey(ssc: StreamingContext): Unit = {
        val intervalMills = 10 * 1000 // 每隔10秒扫描一次消息是否存在
        var isStop = false
        while (!isStop) {
          isStop = ssc.awaitTerminationOrTimeout(intervalMills)
          if (!isStop ) {
            println("2秒后开始关闭sparstreaming程序.....")
            Thread.sleep(2000)
            ssc.stop(stopSparkContext=false, stopGracefully=true)
          }
        }
      }
```
写入文件
![21711-e57611n3n7h.png](http://www.sukidesu.top/usr/uploads/2019/12/1077138185.png)

> 注意：使用Zeppelin 运行streaming时，需要使用ssc.stop(stopSparkContext=false, stopGracefully=true) 进行关闭。

**2.采用独立应用程序方式创建文件流**
使用sbt进行打包
## 7.4.2 套接字流
Spark Streaming 可以通过 Socket 端口监听并接收数据，然后进行相应处理。
**1.Socket 工作原理**
服务器端先初始化 Socket，然后与端口绑定（Bind），对端口进行监听（Listen），调用 accept()方法进入阻塞状态，等待客户端连接。客户端初始化一个 Socket，然后连接服务器（Connect），如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。
![16913-3ohm801eiac.png](http://www.sukidesu.top/usr/uploads/2019/12/4272934152.png)
**2.使用套接字流作为数据源**
套接字的流程与TextFile差不多

    import org.apache.spark.storage.StorageLevel
    ...
    val lines = ssc.socketTextStream(ip,port,StorageLevel.MEMORY_AND_DISK_SER)
    ...

# 7.5高级数据源
1.添加相关jar
首先需要spark-streaming-kafka-0-8_2.11-2.4.5.jar放入spark目录下。
![62790-picji8j2uvo.png](http://www.sukidesu.top/usr/uploads/2020/03/323644587.png)
[官方API:http://spark.apache.org/docs/2.3.1/streaming-kafka-0-10-integration.html][1]
同时需要kafka全部的jar包，运行时指定jar路径即可

    spark-shell --jars /opt/spark/spark-2.3.1-bin-hadoop2.7/jars/*:/opt/kafka_2.11-0.11.0.0/libs/*

在zeppelin中使用，需要在Interpreter中指定参数，然后重启。
导入kafka包无问题即可.




  [1]: http://spark.apache.org/docs/2.3.1/streaming-kafka-0-10-integration.html01postpublish011105530362bcbea55bdc1043861dbd3335fd724