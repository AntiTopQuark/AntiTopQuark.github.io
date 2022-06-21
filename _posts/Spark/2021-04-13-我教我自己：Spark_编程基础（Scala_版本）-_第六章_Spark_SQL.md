---
layout: post
cid: 39
title: 我教我自己：Spark 编程基础（Scala 版本）- 第六章 Spark SQL
slug: 39
date: 2021/04/13 17:32:41
updated: 2021/04/13 17:32:41
status: publish
author: AntiTopQuark
categories: 
  - Spark
  - 大数据
tags: 
  - 反射
  - 类型
  - rdd
  - sql
  - 逻辑
  - 函数
  - 程序
  - 方法
  - 编译
  - 语句
  - 数据库
  - 用户
  - 错误
  - 进程
  - 对象
  - Python
  - 文件
  - spark
  - map
  - 数据
  - 操作
  - 记录
  - 框架
  - class
  - 过程
  - 架构
  - apache
  - user
  - p
  - 表
  - 创建
  - select
  - 语法
  - builder
  - 开发
  - 处理
  - 文本
  - val
  - dataframe
  - hive
  - 机器学习
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

# 6.1 Spark SQL简介
## 6.1.1 从 Shark 说起
Hive 中 SQL 查询转化成 MapReduce 作业的过程
![Hive 中 SQL 查询转化成 MapReduce 作业的过程](http://www.sukidesu.top/usr/uploads/2019/12/1640965778.png)
Shark 提供了类似 Hive 的功能，与 Hive 不同的是，Shark 把 SQL 语句转换成 Spark 作业，而不是 MapReduce 作业。为了实现与 Hive 兼容，Shark 重用了 Hive 中的 HiveQL 解析、逻辑执行计划翻译、执行计划优化等逻辑，可以近似认为，Shark 仅将物理执行计划从 MapReduce 作业替换成了 Spark 作业，也就是通过 Hive 的 HiveQL 解析功能，把 HiveQL 翻译成 Spark 上的 RDD 操作。Shark 的出现，使得 SQL-on-Hadoop 的性能比 Hive 有了 10～100 倍的提高。
![图 6-2 Shark 直接继承了 Hive 的各个组件](http://www.sukidesu.top/usr/uploads/2019/12/4160131124.png)
Shark 的设计导致了两个问题：
一是执行计划优化完全依赖于 Hive，不方便添加新的优化策略；
二是因为 Spark 是线程级并行，而 MapReduce 是进程级并行，因此，Spark 在兼容 Hive 的实现上存在线程安全问题，导致 Shark 不得不使用另外一套独立维护的、打了补丁的 Hive 源码分支。 Shark 的实现继承了大量的 Hive 代码，因而给优化和维护带来了大量的麻烦，特别是基于 MapReduce 设计的部分，成为整个项目的瓶颈。因此，在 2014 年的时候，Shark 项目中止，并转向 Spark SQL 的开发。

## 6.1.2 Spark SQL 架构

Spark SQL 的架构如图所示，在 Shark 原有的架构上重写了逻辑执行计划的优化部分，解决了 Shark 存在的问题。
![91477-093hnicjya6f.png](http://www.sukidesu.top/usr/uploads/2019/12/3731878410.png)
Spark SQL 在 Hive 兼容层面仅依赖 HiveQL 解析和 Hive 元数据，也就是说，从 HiveQL 被解析成抽象语法树（Abstract Syntax Tree,AST）起，剩余的工作全部都由 Spark SQL 接管，即执行计划生成和优化都由 Catalyst（函数式关系查询优化框架）负责。
Spark SQL 增加了 **DataFrame（即带有 Schema 信息的 RDD）**，使用户可以在 Spark SQL 中执行 SQL 语句，数据既可以来自 RDD，也可以来自 Hive、HDFS、Cassandra 等外部数据源，还可以是 JSON 格式的数据。

## 6.1.3 为什么退出Spark SQL
首先，Spark SQL 可以提供 DataFrame API，可以对内部和外部各种数据源执行各种关系操作；其次，可以支持大量的数据源和数据分析算法，组合使用 Spark SQL 和 Spark MLlib，可以融合传统关系数据库的结构化数据管理能力和机器学习算法的数据处理能力，有效满足各种复杂的应用需求。

# 6.2 DataFrame 概述

DataFrame 是一种以 RDD 为基础的分布式数据集，提供了详细的结构信息，就相当于关系数据库的一张表。
![图 6-5 DataFrame 与 RDD 的区别](http://www.sukidesu.top/usr/uploads/2019/12/3750245633.png)

# 6.3 DataFrame的创建
从 Spark2.0 以上版本开始，Spark 使用全新的 SparkSession 接口替代 Spark1.6 中的 SQLContext 及 HiveContext 接口，来实现其对数据加载、转换、处理等功能。SparkSession 实现了 SQLContext 及 HiveContext 所有功能。 
SparkSession 支持从不同的数据源加载数据，以及把数据转换成 DataFrame，并且支持把 DataFrame 转换成 SQLContext 自身的表，然后使用 SQL 语句来操作数据。SparkSession 亦提供了 HiveQL 以及其他依赖于 Hive 的功能的支持。 
可以通过如下语句创建一个 SparkSession 对象： 

    import org.apache.spark.sql.SparkSession 
    val spark=SparkSession.builder().getOrCreate() 

实际上，在启动进入 spark-shell 以后，spark-shell 就默认提供了一个 SparkContext 对象（名称为 sc）和一个 SparkSession 对象（名称为 spark），因此，也可以不用自己声明一个 SparkSession 对象，而是直接使用 spark-shell 提供的 SparkSession 对象，即 spark。
在创建 DataFrame 之前，为了支持 RDD 转换为 DataFrame 及后续的 SQL 操作，需要通过 import 语句（即 `import spark.implicits._`）导入相应的包，启用隐式转换。 在创建 DataFrame 时，可以使用 spark.read 操作，从不同类型的文件中加载数据创建 DataFrame，例如：

- spark.read.json（「people.json」）：读取 people.json 文件创建 DataFrame；在读取本地文件或 HDFS 文件时，要注意给出正确的文件路径； 
- spark.read.parquet（「people.parquet」）：读取 people.parquet 文件创建 DataFrame；  
- spark.read.csv（「people.csv」）：读取 people.csv 文件创建 DataFrame。 

或者也可以使用如下格式的语句： 

- spark.read.format（「json」）.load（「people.json」）：读取 people.json 文件创建 DataFrame； 
- spark.read.format（「csv」）.load（「people.csv」）：读取 people.csv 文件创建 DataFrame； 
- spark.read.format（「parquet」）.load（「people.parquet」）：读取 people.parquet 文件创建 DataFrame。

需要指出的是，从文本文件中读取数据创建 DataFrame，无法直接使用上述类似的方法，需要使用后面介绍的***「从 RDD 转换得到 DataFrame」***。

# 6.4 DataFrame 的保存
可以使用 spark.write 操作，把一个 DataFrame 保存成不同格式的文件。例如，把一个名称为 df 的 DataFrame 保存到不同格式文件中，方法如下：
- df.write.json("people.json")； 
- df.write.parquet("people.parquet")； 
- df.write.csv("people.csv")； 
或者也可以使用如下格式的语句： 
- df.write.format("json").save("people.json")； 
- df.write.format ("csv").save("people.csv")； 
- df.write.format ("parquet").save("people.parquet")；
注意，上述操作只简单给出了文件名称，在实际进行上述操作时，一定要给出正确的文件路径
# 6.5 DataFrame的常用操作
DataFrame 创建好以后，可以执行一些常用的 DataFrame 操作，包括 printSchema()、select()、filter()、groupBy()和 sort()等。
- printSchema() 
可以使用 printSchema()操作，打印出 DataFrame 的模式（Schema）信息。
![09352-cved22znzcp.png](http://www.sukidesu.top/usr/uploads/2019/12/3159705279.png)
- select() 
select()操作的功能，是从 DataFrame 中选取部分列的数据。如图 6-8 所示，select()操作选取了 name 和 age 这两个列，并且把 age 这个列的值增加 1。
![77775-hb5cqmi10p.png](http://www.sukidesu.top/usr/uploads/2019/12/2587666233.png)
select()操作还可以实现对列名称进行重命名的操作。如图 6-9 所示，name 列名称被重命名为 username。
![12427-7utcx1h1bm9.png](http://www.sukidesu.top/usr/uploads/2019/12/3806939783.png)
- filter() 
filter()操作可以实现条件查询，找到满足条件要求的记录。如图 6-10 所示，df.filter(df（「age」）>20)用于查询所有 age 字段大于 20 的记录。
![38077-zxsrpsttf1m.png](http://www.sukidesu.top/usr/uploads/2019/12/48163188.png)
- groupBy()
groupBy()操作用于对记录进行分组。如图 6-11 所示，可以根据 age 字段进行分组，并对每个分组中包含的记录数量进行统计。
![45825-9045d3t3ama.png](http://www.sukidesu.top/usr/uploads/2019/12/2080887.png)
- sort() 
sort()操作用于对记录进行排序。如图 6-12 所示，df.sort(df（「age」）.desc)表示根据 age 字段进行降序排序。df.sort(df（「age」）.desc，df（「name」）.asc)表示根据 age 字段进行降序排序，当 age 字段的值相同时，再根据 name 字段进行升序排序。
![78490-lupjlnepzx.png](http://www.sukidesu.top/usr/uploads/2019/12/1865018695.png)
# 6.6 从 RDD 得到 DataFrame
## 6.6.1 利用反射机制推断
定义 case class，作为 RDD 的 schema
直接通过 RDD.toDF 将 RDD 转换为 DataFrame

    import org.apache.spark.sql.SQLContext
    import org.apache.spark.sql.Row
    case class User(userID: Long, gender: String, age: Int, occupation: String, zipcode: Int)
    val usersRdd = sc.textFile("/tmp/ml-1m/users.dat")
    val userRDD = usersRdd.map(_.split("::")).map(p => User(p(0).toLong, p(1).trim,p(2).toInt, p(3), p(4).toInt))
    val userDataFrame = userRDD.toDF()
    userDataFrame.take(10)
    userDataFrame.count()

## 6.6.2 使用编程方式定义 RDD 模式
定义RDD schema（由StructField/StructType构成）
使用SQLContext.createDataFrame生成DF

    import org.apache.spark.sql.{SaveMode, SQLContext, Row}
    import org.apache.spark.sql.types.{StringType, StructField, StructType}
    val schemaString = "userID gender age occupation zipcode"
    val schema = StructType(schemaString.split(" ").map(fieldName =>
    StructField(fieldName, StringType, true)))
    val userRDD2 = usersRdd.map(_.split("::")).map(p => Row(p(0), p(1).trim, p(2).trim,
    p(3).trim, p(4).trim))
    val userDataFrame2 = sqlContext.createDataFrame(userRDD2, schema)
    userDataFrame2.take(10)
    userDataFrame2.count()
    userDataFrame2.write.mode(SaveMode.Overwrite).json("/tmp/user.json")
    userDataFrame2.write.mode(SaveMode.Overwrite).parquet("/tmp/user.parquet")

# 6.7 使用 Spark SQL读写数据库

## 6.7.1使用JDBC链接数据库

spark.read.format（「jdbc」）操作可以实现对 MySQL 数据库的读取。执行以下命令连接数据库，读取数据并显示：

![73846-y157stezwdj.png](http://www.sukidesu.top/usr/uploads/2019/12/4030274327.png)

## 6.7.2使用HIVE读写数据
![40982-buicnegd7us.png](http://www.sukidesu.top/usr/uploads/2019/12/2774010447.png)

# Spark Dataset
相对于RDD，Dataset提供了强类型支持，也是在RDD的每行数据加了类型约束

假设RDD中的两行数据长这样
![51320-ebu9o435oy.png](http://www.sukidesu.top/usr/uploads/2020/03/3985560401.png)

那么Dataset中的数据长这样

![27071-0grtyrg371ou.png](http://www.sukidesu.top/usr/uploads/2020/03/2470841108.png)
或者长这样（每行数据是个Object）
![70234-ljnfg2n8o5.png](http://www.sukidesu.top/usr/uploads/2020/03/532765120.png)

使用Dataset API的程序，会经过Spark SQL的优化器进行优化（优化器叫什么还记得吗？）

目前仅支持Scala、Java API，尚未提供Python的API（所以一定要学习Scala）

相比DataFrame，Dataset提供了编译时类型检查，对于分布式程序来讲，提交一次作业太费劲了（要编译、打包、上传、运行），到提交到集群运行时才发现错误，实在是想骂人，这也是引入Dataset的一个重要原因。

使用DataFrame的代码中json文件中并没有score字段，但是能编译通过，但是运行时会报异常！如下图代码所示
![86194-5wa45szuhww.png](http://www.sukidesu.top/usr/uploads/2020/03/830022155.png)

而使用Dataset实现，会在IDE中就报错，出错提前到了编译之前
![52932-42pa47qwqvs.png](http://www.sukidesu.top/usr/uploads/2020/03/1610733829.png)

RDD转换DataFrame后不可逆，但RDD转换Dataset是可逆的（这也是Dataset产生的原因）。

作者：dingyuanpu
链接：https://www.jianshu.com/p/77811ae29fdd
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。