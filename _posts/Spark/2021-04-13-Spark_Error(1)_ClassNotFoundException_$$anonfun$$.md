---
layout: post
cid: 40
title: Spark Error(1):ClassNotFoundException:$$anonfun$$
slug: 40
date: 2021/04/13 17:34:00
updated: 2021/04/13 17:34:47
status: publish
author: AntiTopQuark
categories: 
  - Spark
  - BugS
tags: 
  - 程序
  - spark
  - jvm
  - class
  - 过程
  - master
  - val
  - 问题
  - local
  - cluster
  - setmaster
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


在使用idea + maven 进行spark app 本地(windows)开发的过程中，遇到了
```
java.lang.ClassNotFoundException:$$anonfun$1 .

val conf = new SparkConf
conf.setMaster(spark://<ip>:7077) 
```

 

经过查阅资料，发现引起这个问题的原因是因为jvm 没有把最新的代码全部读取。因为我是在本地开发的，所以最新代码在local, 而master 我是指向了spark cluster, 因而导致在spark cluster 运行时 jvm 没有得到最新的代码。

 

解決方案有2个:

1.  在local , setMaster(local[*]) 或者 setMaster(local[2]) // 2表示多少核可以使用

2.  在cluster, 把程序打包上传server ,然后通过spark-submit 执行。如：
```
spark-submit --class <main class>  --master spark://<ip>:7077 /path/to/app.jar
```