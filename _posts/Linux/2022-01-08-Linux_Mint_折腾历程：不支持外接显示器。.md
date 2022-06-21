---
layout: post
cid: 142
title: Linux Mint 折腾历程：不支持外接显示器。
slug: 142
date: 2022/01/08 21:48:48
updated: 2022/01/08 21:48:48
status: publish
author: AntiTopQuark
categories: 
  - Linux
tags: 
  - 系统
  - 用户
  - 文件
  - 内核
  - 问题
  - deb
  - secure
  - kernel
  - boot
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


找了很久，最后发现是Linux Kernel的问题。
我的Kernel的版本是5.4.x，更新到5.7以上就好了。
参考来自https://blog.csdn.net/a586351/article/details/107193070?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2%7Eall%7Efirst_rank_v2%7Erank_v25-6-107193070.nonecase&utm_term=ubuntu%20wifi%20%E5%B0%8F%E6%96%B0pro13

------------


默认安装好系统以后是100%的缩放，而你只能调整200%或300%。亮度无法调节/缩放无法调整/无法使用HDMI扩展显示屏，其实都是一个问题导致的——内核版本
现在系统默认是5.4的内核，我们可以在ubuntu kernel查到最新版本是5.8rc，当然作为普通用户我建议大家更新到最新的稳定版就好了，以我编写教程的时间为例，是5.7.7
（目前版本已经是5.15了）
https://kernel.ubuntu.com/~kernel-ppa/mainline/
![](http://www.sukidesu.top/usr/uploads/2022/01/3962595112.png)
点开网页滑到底，选择你的系统版本，我是AMD64（intel用户你也是选AMD64）
![](http://www.sukidesu.top/usr/uploads/2022/01/462828668.png)
三个generic的deb，加一个_all的deb，下载到一个文件夹中。
![](http://www.sukidesu.top/usr/uploads/2022/01/1890624129.png)
注意文件夹中不要有其他的deb文件了
下载好以后，在终端执行sudo dpkg -i *.deb

安装好后，reboot即可
重启的时候记得在BIOS里面关闭secure boot！
记得在BIOS里面关闭secure boot！
关闭secure boot！
！！！
重启后查看你的内核uname -r，是不是已经更新了呢？

现在，你可以调整你的亮度/缩放/外接显示器了！
开始享受你的coding时光吧
针对ubuntu 18.04升级5.7+内核的问题
1. 查看当前内核
![](http://www.sukidesu.top/usr/uploads/2022/01/248127518.png)
2. 下载你要升级的内核
![](http://www.sukidesu.top/usr/uploads/2022/01/986571738.png)
3. 升级内核
![](http://www.sukidesu.top/usr/uploads/2022/01/2475133907.png)
4. 显示 done，即完成
5. 重启即可启用新内核