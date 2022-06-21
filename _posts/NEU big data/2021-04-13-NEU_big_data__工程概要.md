---
layout: post
cid: 33
title: NEU big data: 工程概要
slug: 33
date: 2021/04/13 17:13:51
updated: 2021/04/13 17:13:51
status: publish
author: AntiTopQuark
categories: 
  - NEU big data
tags: 
  - 系统
  - 引擎
  - 存储
  - 事务
  - 界面
  - 用户
  - 状态
  - spark
  - 任务
  - 数据
  - 框架
  - apache
  - web
  - g
  - 开发
  - 处理
  - scala
  - hive
  - 计算
  - 机器学习
  - 编程
  - livy
  - zeppelin
  - lake
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


### 一、Zeppelin
Apache Zeppelin提供了web版的类似ipython的notebook，用于做数据分析和可视化。背后可以接入不同的数据处理引擎，包括spark, hive, tajo等，原生支持scala, java, shell, markdown等。
在 Zeppelin 中还可以完成机器学习的数据预处理、算法开发和调试、算法作业调度的工作，同时，Zeppelin 还提供了单机 Docker、分布式、K8s、Yarn 四种系统运行模式.

![v2-d1e9f0f2c84775eab759eab70adf39d8_r.jpg][1]

### 二、Livy
除了 Apache Spark 本身提供的 spark-submit、spark-shell 和 ThriftServer 之外， Apache Livy 提供了另一种与 Spark 集群交互的方式，通过 REST 接口。
此外，Apache Livy 支持同时维护多个session。
可以通过 REST 接口、Java/Scala 库和 Apache Zeppelin 访问 Apache Livy。

###三、Airflow
 Airflow是一个可编程，调度和监控的工作流平台，基于有向无环图(DAG)，airflow可以定义一组有依赖的任务，按照依赖依次执行。airflow提供了丰富的命令行工具用于系统管控，而其web管理界面同样也可以方便的管控调度任务，并且对任务运行状态进行实时监控，方便了!系统的运维和管理。
[564309-20180809182421066-723069403.jpg][2]

###四、Delta Lake
Delta Lake是Spark计算框架和存储系统之间带有Schema信息数据的存储中间层。它给Spark带来了三个最主要的功能：第一，Delta Lake使得Spark能支持数据更新和删除功能；第二，Delta Lake使得Spark能支持事务；第三，支持数据版本管理，运行用户查询历史数据快照。






  [1]: http://www.sukidesu.top/usr/uploads/2019/10/1441785074.jpg
  [2]: http://www.sukidesu.top/usr/uploads/2019/10/4263769340.jpg