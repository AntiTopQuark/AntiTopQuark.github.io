---
layout: post
cid: 32
title: NEU big data : install cuda and cudnn
slug: 32
date: 2021/04/13 17:12:44
updated: 2021/04/13 17:12:44
status: publish
author: AntiTopQuark
categories: 
  - NEU big data
tags: 
  - 变量
  - 方法
  - 文件
  - 笔者
  - 框架
  - 过程
  - 服务器
  - 深度学习
  - 测试
  - 环境
  - 安装
  - linux
  - 计算
  - source
  - install
  - 问题
  - cuda
  - cudnn
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


首先介绍一下cuda 与 cudnn:
CUDA是NVIDIA推出的用于自家GPU的并行计算框架，也就是说CUDA只能在NVIDIA的GPU上运行，而且只有当要解决的计算问题是可以大量并行计算的时候才能发挥CUDA的作用。
cuDNN是一个SDK，是一个专门用于神经网络的加速包，注意，它跟我们的CUDA没有一一对应的关系，即每一个版本的CUDA可能有好几个版本的cuDNN与之对应，但一般有一个最新版本的cuDNN版本与CUDA对应更好。


<!--more-->


####  1.首先验证你是否有nvidia的显卡（[查看你是否有支持gpu的显卡][1]）:

    lspci | grep -i nvidia 
结果如图所示：
![查看是否有nvidia的显卡][2]

#### 2.查看你的linux发行版本（主要是看是64位还是32位的）：

    uname -m && cat /etc/*release
结果：
![查看linux发行版本][3]
#### 3.看一下gcc的版本：

    gcc –v

结果：
![gcc版本][4]

#### 4.查看NVIDIA显卡的驱动版本

    cat /proc/driver/nvidia/version

结果：
![查看显卡驱动版本][5]

#### 5.现在对应版本的CUDA Toolkit
 在NVIDIA官网下载对应的安装文件，建议选择.run 文件,再上传到服务器上（毕竟，用服务器上外网需要登陆网关）
 选择对应的选项

![下载cuda][6]

![下载链接][7]
**方法一：**本地直接打开对应的连接,例如：http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run ，上传到服务器后，然后运行命令：
    sudo sh cuda_10.1.243_418.87.00_linux.run
**方法二:**登陆网关后，
直接在服务器执行上图命令

注意：现在安装的版本是最新版本（10.1.update2),如果想安装之前的版本，可以打开： [https://developer.nvidia.com/cuda-toolkit-archive][8] 同时，要注意显卡与cuda的对应版本： [https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#major-components][9]

 ![TIM截图20191025141042.jpg][10]

安装过程如图所示：
1）输入accept
![TIM截图20191025152101.jpg][11]

2）去掉driver 否则会安装失败
![TIM截图20191025153154.jpg][12]

3）summary中会告诉我们cuda的安装路径以及环境变量的配置方法,不过由于我们后面要配置cudnn,所以可以等以后再配置。
![20191025152909.png][13]


#### 6.下载cudnn
https://developer.nvidia.com/rdp/cudnn-archive 打开页面，最好选择与cuda对应的版本下载。
下载时，可能需要注册，不过放心，是免费的，就是邮箱会遭到产品推荐和深度学习讲座的轰炸而已。

![TIM截图20191025142141.jpg][14]

注：下载可能较慢，如果有需要，可以到ftp下载（写文章时，还没有上传上去）

1）解压文件

    tar -xzvf cudnn-10.1-linux-x64-v7.6.3.30.tgz

2)将以下文件复制到CUDA Toolkit目录中，然后更改文件权限

    sudo cp cuda/include/cudnn.h /usr/local/cuda/include
    sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

3)在环境变量中添加

    export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
    export CUDA_HOME=/usr/local/cuda

记得 source /etc/profile

####7. 测试cuda与cudnn
首先测试cuda:

    cd /usr/local/cuda-10.1/samples/1_Utilities/deviceQuery
    sudo make
    sudo ./deviceQuery
测试时注意cuda版本，笔者这里是10.1
当运行的最后一行出现 Result=PASS 时，说明运行成功
![运行结果][15]

到目前为止，说明我们的cuda和cudnn已经安装成功了。
考虑到通用性，这里笔者使用了 pip3 安装了pytorch 最新版本（1.3）进行测试
请提前安装好python3 与pip3
#### 8.安装pytorch torchvision 包
在pytorch官网上可以学到pip3命令

![TIM截图20191025190022.jpg][16]

但是因为是从外网下载，实在是太慢了。（甚至有点想偷偷装个酸酸乳）
所以笔者提前下载了 .whl 文件 通过pip3 安装，同时由于可能需要从外网安装numpy，也会过慢 ，所以这里使用a'li'yua源 [https://mirrors.tuna.tsinghua.edu.cn/][17] 为什么不直接使用国内源安装torch呢？因为清华源只有torch==0.1.12!!!最低版本的。而且中科大源和清华源合并了。阿里源也没找到torch，所以这里我们只用国内源安装numpy;

安装 torch:

    pip3 install torch-1.3.0-cp36-cp36m-manylinux1_x86_64.whl -i https://mirrors.aliyun.com/pypi/simple

安装 torchvision:

    pip3 install torchvision-0.4.1-cp36-cp36m-manylinux1_x86_64.whl -i https://mirrors.aliyun.com/pypi/simple

测试 GPU:

![TIM截图20191025191136.jpg][18]

如果看到true,说明整个环境安装成功。可喜可贺，可喜可贺

  [1]: http://developer.nvidia.com/cuda-gpus
  [2]: http://www.sukidesu.top/usr/uploads/2019/10/620974446.jpg
  [3]: http://www.sukidesu.top/usr/uploads/2019/10/4141913285.jpg
  [4]: http://www.sukidesu.top/usr/uploads/2019/10/2421948948.jpg
  [5]: http://www.sukidesu.top/usr/uploads/2019/10/2064247741.jpg
  [6]: http://www.sukidesu.top/usr/uploads/2019/10/2136479571.jpg
  [7]: http://www.sukidesu.top/usr/uploads/2019/10/595507265.jpg
  [8]: https://developer.nvidia.com/cuda-toolkit-archive
  [9]: https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html#major-components
  [10]: http://www.sukidesu.top/usr/uploads/2019/10/1871594082.jpg
  [11]: http://www.sukidesu.top/usr/uploads/2019/10/3441123737.jpg
  [12]: http://www.sukidesu.top/usr/uploads/2019/10/639075412.jpg
  [13]: http://www.sukidesu.top/usr/uploads/2019/10/2467887465.png
  [14]: http://www.sukidesu.top/usr/uploads/2019/10/1790805880.jpg
  [15]: http://www.sukidesu.top/usr/uploads/2019/10/2640236689.jpg
  [16]: http://www.sukidesu.top/usr/uploads/2019/10/4233996891.jpg
  [17]: https://mirrors.aliyun.com/pypi/simple
  [18]: http://www.sukidesu.top/usr/uploads/2019/10/1314637619.jpg