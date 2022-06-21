---
layout: post
cid: 132
title: HIVE是如何进行update和delete操作的？
slug: 132
date: 2021/12/01 14:49:09
updated: 2021/12/01 14:49:09
status: publish
author: AntiTopQuark
categories: 
  - 大数据
tags: 
  - type
  - 类型
  - 程序
  - 内存
  - 前言
  - 系统
  - 存储
  - 事务
  - 查询
  - 文件
  - hdfs
  - 分区
  - hbase
  - 数据
  - 记录
  - 编码
  - 索引
  - update
  - 表
  - 创建
  - key
  - 问题
  - hash
  - orc
  - stripe
customSummary: 
mathjax: auto
noThumbInfoStyle: default
outdatedNotice: no
reprint: standard
thumb: 
thumbChoice: default
thumbDesc: 
thumbSmall: 
thumbStyle: default
---



### 前言
Hive从0.14版本开始支持事务和行级更新,支持行级insert、update、delete等操作。

但是众所周知，HIVE是基于HDFS的，而HDFS是不支持修改操作的，HDFS只支持文件append操作，那么HIVE是如何进行update和delete操作的呢？

其实从HIVE在配置的时候就可以看出来是如何做到的。要执行update和delete的表必须支持ACID，即
- 必须使用ORC文件格式
- 表必须被分桶了

### ORC 文件存储格式
ORC其实是一种列式存储，引用其他博主的介绍：和Parquet类似，ORC文件也是以二进制方式存储的，所以是不可以直接读取，ORC文件也是自解析的，它包含许多的元数据，这些元数据都是同构ProtoBuffer进行序列化的。ORC的文件结构如下图，其中涉及到如下的概念：

- ORC文件：保存在文件系统上的普通二进制文件，一个ORC文件中可以包含多个stripe，每一个stripe包含多条记录，这些记录按照列进行独立存储，对应到Parquet中的row group的概念。
- 文件级元数据：包括文件的描述信息PostScript、文件meta信息（包括整个文件的统计信息）、所有stripe的信息和文件schema信息。
- stripe：一组行形成一个stripe，每次读取文件是以行组为单位的，一般为HDFS的块大小，保存了每一列的索引和数据。
- stripe元数据：保存stripe的位置、每一个列的在该stripe的统计信息以及所有的stream类型和位置。
- row group：索引的最小单位，一个stripe中包含多个row group，默认为10000个值组成。
- stream：一个stream表示文件中一段有效的数据，包括索引和数据两类。索引stream保存每一个row group的位置和统计信息，数据stream包括多种类型的数据，具体需要哪几种是由该列类型和编码方式决定。
![](http://www.sukidesu.top/usr/uploads/2021/12/174708013.png)
### 分桶
#### 为什么要分桶？
单个分区或者表中的数据量越来越大，当分区不能更细粒的划分数据时，所以会采用分桶技术将数据更细粒度的划分和管理。
#### 分桶的意义：
1、为了保存分桶查询结果的分桶结构（数据已经按照分桶字段进行了hash散列）
2、分桶表数据进行抽样和JOIN时可以提高MR程序效率


### 综上所述
结合这两点，我们就可以猜测一下，每个orc文件对应着一个桶，由于是列存，修改或者删除的就可以通过append操作来处理了。

### 另一个问题，HBase是如何处理这个问题的？
其实也很简单，Hbase都有时间戳，旧的数据会在flush的时候被覆盖。

HBase表中的数据当存放到HDFS中时，在HDFS看来，已经可以简单的理解成key-value对，其中key可以理解成是由：rowkey+column family+column qualifier+timestamp+type 组成。HBase 对新增的数据以及要更新的数据（理解成key-value对），都直接先写入MemStore结构中，其实就是 LSM（The Log-Structured Merge-Tree） ，Memstore 就是上边的 Memtable，HFiles 就是上边的 SSTables (HBase权威指南 上 也有明确)，MemStore是完全的内存结构，且是key有序的。当MemStore达到一定大小后，该MemStore一次性从内存flush到HDFS中（磁盘中），生成一个HFile文件，HFile文件同样是key有序的，并且是持久化的位于HDFS系统中的。通过这种机制，HBase对表的所有的插入和更新都转换成对HDFS的HFile文件的创建。

你可能会迅速的想到，那查询怎么办？

是的，这种方式解决了插入和更新的问题，而查询就变得相对麻烦。而这也正是HBase设计之初的想法：以查询性能的下降来换取更新性能的提升。