---
layout: post
cid: 34
title: NEU big data: stop-all.sh失效
slug: 34
date: 2021/04/13 17:15:01
updated: 2021/04/13 17:15:01
status: publish
author: AntiTopQuark
categories: 
  - NEU big data
  - BugS
tags: 
  - 变量
  - 方法
  - 系统
  - 用户
  - 错误
  - 进程
  - namenode
  - 文件
  - master
  - 环境
  - dfs
  - linux
  - 问题
  - hadoop
  - datanode
  - pids
  - start-all
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


长时间运行hadoop之后，如果运行stop-all.sh，会发现有以下错误：

    no jobtracker to stop
    hadoop3: no tasktracker to stop
    hadoop2: no tasktracker to stop
    no namenode to stop
    hadoop2: no datanode to stop
    hadoop3: no datanode to stop
    hadoop1: no secondarynamenode to stop

这个时候访问hadoop依然有效，查看文件系统，通过50070端口依然能访问，start-all后再stop-all也没有任何效果，等于这个时候完全无法控制hadoop了。

出现这个问题的最常见原因是hadoop在stop的时候依据的是datanode上的mapred和dfs进程号。而默认的进程号保存在/tmp下，linux默认会每隔一段时间（一般是一个月或者7天左右）去删除这个目录下的文件。因此删掉hadoop-hadoop-jobtracker.pid和hadoop-hadoop-namenode.pid两个文件后，namenode自然就找不到datanode上的这两个进程了。

另外还有两个原因可能引起这个问题：

1：环境变量 $HADOOP_PID_DIR 在你启动hadoop后改变了

2：用另外的用户身份执行stop-all

解决方法：

1：永久解决方法，修改$HADOOP_HOME/conf/hadoop-env.sh里边，去掉export HADOOP_PID_DIR=/var/hadoop/pids的#号，创建/var/hadoop/pids或者你自己指定目录

发现问题后的解决方法：

1：这个时候通过脚本已经无法停止进程了，不过我们可以手工停止，方法是到各mfs master和各datanode执行ps -ef | grep java | grep hadoop找到进程号强制杀掉，然后在master执行start-all脚本重新启动，就能正常启动和关闭了。
