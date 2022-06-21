---
layout: post
cid: 38
title: 我教我自己：Spark 编程基础（Scala 版本）- 第五章 RDD编程
slug: 38
date: 2021/04/13 17:31:00
updated: 2021/04/13 17:31:56
status: publish
author: AntiTopQuark
categories: 
  - Spark
  - 大数据
tags: 
  - 类型
  - func
  - rdd
  - 函数
  - 程序
  - 方法
  - 系统
  - 文件
  - spark
  - 数据
  - 操作
  - 过程
  - 参数
  - 表
  - scala
  - val
  - 转换
  - 计算
  - 编程
  - usr
  - local
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


# 5.1 RDD编程基础
## 5.1.1 RDD创建
Spark 采用 textFile()方法来从文件系统中加载数据创建 RDD，该方法把文件的 URI 作为参数，这个 URI 可以是本地文件系统的地址、分布式文件系统 HDFS 的地址，或者是 Amazon S3 的地址等。
（1）通过文件系统创建RDD

    val lines = sc.textFile("file:///usr/local/spark/mycode/rdd/word.txt")

(2)通过并行集合创建RDD
可以调用 SparkContext 的 parallelize 方法，从一个已经存在的集合（数组）上创建 RDD（见图 5-2），命令如下： 

    scala> val array = Array(1,2,3,4,5) 
    scala> val rdd = sc.parallelize(array) 

或者，也可以从列表中创建，命令如下： 

    scala> val list = List(1,2,3,4,5) 
    scala> val rdd = sc.parallelize(list)

![从数组创建 RDD 示意图](http://www.sukidesu.top/usr/uploads/2019/12/1577438904.png)

## 5.1.2 RDD操作

RDD 操作包括两种类型，即转换（Transformation）操作和行动（Action）操作。 
1.转换操作 
对于 RDD 而言，每一次转换操作都会产生不同的 RDD，供给下一个操作使用。RDD 的转换过程是惰性求值的，也就是说，整个转换过程只是记录了转换的轨迹，并不会发生真正的计算，只有遇到行动操作时，才会触发「从头到尾」的真正的计算。表 5-1 给出了常用的 RDD 转换操作 API，其中很多操作都是高阶函数，比如，filter（func）就是一个高阶函数，这个函数的输入参数 func 也是一个函数。

![常用的 RDD 转换操作 API](http://www.sukidesu.top/usr/uploads/2019/12/2971980424.png)

2.行动操作
行动操作是真正触发计算的地方。Spark 程序只有执行到行动操作时，才会执行真正的计算，从文件中加载数据，完成一次又一次转换操作，最终，完成行动操作得到结果。表 5-2 列出了常用的 RDD 行动操作 API。



![34550-f40nut4084k.png](http://www.sukidesu.top/usr/uploads/2019/12/507978149.png)

## 5.1.3 持久化
