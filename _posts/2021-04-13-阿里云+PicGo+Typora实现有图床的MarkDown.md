---
layout: post
cid: 80
title: 阿里云+PicGo+Typora实现有图床的MarkDown
slug: 80
date: 2021/04/13 20:37:55
updated: 2021/04/13 20:37:55
status: publish
author: AntiTopQuark
categories: 
  - 默认分类
tags: 
  - 类型
  - 前言
  - 系统
  - 存储
  - 界面
  - 用户
  - 对象
  - 文件
  - 任务
  - 数据
  - 笔记
  - 流程
  - 过程
  - 安装
  - 软件
  - typora
  - oss
  - 步骤
  - picgo
  - 点击
  - 问题
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


[TOC]

## 前言

写MarkDown笔记的时候，有很多很好用的软件，比如Typora和Mweb等等。Mweb是付费的，128￥，对于我来说还是挺贵的。Typora是免费的，也十分好用，之前发现了它有图床的功能。直接在Typora里面粘贴图片，可以自动上传到图床，很适合我这种一篇文章发送到多个平台的作者。这里写一个简答的教程。

## 阿里云OSS

### 步骤一:来到阿里云对象存储OSS界面

阿里云对象存储OSS界面：https://www.aliyun.com/product/oss，开通服务后，点击OSS控制台。

可以先看下价格，0.12元/GB/月的价格个人用还是蛮划算的。

![OSS价格](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117174229.png)

### 步骤二：创建Bucket，填写名称等相关信息

存储类型选择标准存储比较好，如果访问量大的话，其他类型的存储可能会导致速度较慢。

需要注意的是读写权限要选择`公有读`，否则在读取数据的时候，需要填写秘钥信息。图片就不能给别人看了。

其他选项根据自己的需要来好。

![创建Bucket1](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117174848.png)

![创建Bucket2](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117175000.png)

![创建Bucket3](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117175025.png)

### 步骤三：新建img/ 文件夹

当然，也可以把图片上传到根目录，不过这样就很难进行整理了。推荐新创建一个图片文件夹


![新建文件夹1](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117175556.png)


![新建文件夹2](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117175638.png)

### 步骤四：创建Access Key，这个是用来上传图片的钥匙。注意：不能透露给别人，否则别人就可能上传图片到自己的OSS中。
来到对象存储OSS控制台，点击Access Key

![image-20210117180036562](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117180036.png)

进入到新页面，为了安全，我们选择子用户Access Key。

![image-20210117180124176](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117180124.png)

填写子用户信息,会进行一次短信验证码认证。

![image-20210117180326715](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117180326.png)



先点击CheckBox,增加子用户权限

![image-20210117180447005](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117180447.png)

选择OSS权限，增加到子用户，然后点击确定

![image-20210117180556161](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117180556.png)

点击左侧导航栏的用户，进入到用户列表

![image-20210117183342132](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117183342.png)

点击新创建的用户，下拉后创建Access Key

![image-20210117183404842](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117183404.png)

创建AccessKey，会进行一次短信验证

![image-20210117183504513](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117183504.png)

把创建好的AccessKey保存下来，记住一定不要发给陌生人。

![image-20210117183543935](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117183543.png)



## PicGo

PicGo真的太好用了，官网：https://molunerfinn.com/PicGo/

### 步骤一：下载PicGo

来到下载页面：https://github.com/Molunerfinn/picgo/releases

根据系统选择相应的文件，可以选择自己需要的版本，Beta版本不一定稳定，但会有一些新功能。

Windows系统的话选择EXE后缀的文件

![image-20210117184111955](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117184112.png)

### 步骤二：双击exe文件，进行安装。

普通的软件安装过程，大家一定都会，不进行赘述了。

要记得安装路径，后面Typora配置的时候会用到

### 步骤三：进行PicGo配置

从任务栏中打开PicGo，点击图床设置，下拉栏里有阿里云OSS，点击它

![image-20210117184444339](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117184444.png)

### 步骤四： 根据要求填写配置

![image-20210117184519970](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117184520.png)

- KeyID和KeySecret就是我们之前保存好的。

- 存储空间名就是我们的Bucket。

- 存储区域如下图所示，我们首先进入到Bucket管理页面，点击概况，找到我们的Bucket存储地址，填写到城市即可，比如oss-cn-beijing

![image-20210117184740113](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117184852.png)

- 存储路径如果之前有新建文件夹的话，填写即可。

- 也可以将阿里云设置为默认的图床。点击确定，PicGo配置就完成了



### 其他

阿里云的OSS价格还是蛮划算的，要是完全不像花钱的话，也可以使用一些免费的图床。

PicGo还支持的图床有：SM.MS   腾讯云COS  七牛云图床等等

配置的话，根据PicGo的设置要求，填写相应信息即可，这里不再赘述了。

## Typora设置

### 步骤一：下载Typora并安装

Typora的官网是：https://www.typora.io/

安装过程同样不再赘述了。

### 步骤二：打开Typora，并进行配置

打开Typora -> 点击工具栏的“文件” -> 点击“偏好设置” -> 按照下面的顺序进行配置 -> 关闭偏好设置

![image-20210117192504884](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117192504.png)



### 步骤三：测试Typora

找一张图片，直接粘贴在Typora内部，然后查看下链接,如下图所示。

链接是你的阿里云OSS即可

![image-20210117192703892](https://image-bed113224.oss-cn-beijing.aliyuncs.com/img/20210117192703.png)



## 总结

总的配置流程就是这样，还是挺简单的。

如果大家有什么问题，可以在通过微信公众号联系我。

![](C:/Users/pytho/Desktop/20210117193222.png)