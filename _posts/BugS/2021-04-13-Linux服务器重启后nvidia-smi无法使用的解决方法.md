---
layout: post
cid: 60
title: Linux服务器重启后nvidia-smi无法使用的解决方法
slug: 60
date: 2021/04/13 17:54:11
updated: 2021/04/13 17:54:11
status: publish
author: AntiTopQuark
categories: 
  - BugS
tags: 
  - 方法
  - 界面
  - 文件
  - 服务器
  - with
  - install
  - 问题
  - usr
  - nvidia
  - nvidia-smi
  - dkms
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



重启服务器之后就出现连接不上NVIDIA驱动的情况。这个时候pytorch还是可以运行的，但只是在用cpu跑。安装gpu版的pytorch时，也显示已安装。
运行nvidia-smi时

> NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA
> driver. Make sure that the latest NVIDIA driver is installed and
> running.

我们在终端输入 nvcc -V 发现驱动也在。
查找了很多方法之后，发现下面这个最简便，只需要两步，而且还不用重启。
ubuntu:

    sudo apt-get install dkms
    sudo dkms install -m nvidia -v 390.59 

centos:

    yum install dkms
    sudo dkms install -m nvidia -v 430.30


再次输入nvidia-smi时，熟悉的界面就会回来啦。

（虽然使用率显示为99%，但并不影响我们使用）

其中step2中的430.30是NVIDIA的版本号，当你不知道的时候，进入/usr/src目录中，可以看到里面有nvidia文件夹，后缀就是其版本号

cd /usr/src

OK，到此我们就轻松愉快的解决了这个问题。（Yeah!）

