---
layout: post
cid: 57
title: 经典的hash函数&amp;amp;Hash冲突解决方法
slug: 57
date: 2021/04/13 17:49:18
updated: 2021/04/13 17:49:18
status: publish
author: AntiTopQuark
categories: 
  - 算法
tags: 
  - 函数
  - int
  - 数据
  - 表
  - key
  - return
  - 算法
  - hash
  - length
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
# 经典Hash函数
Hash函数是指把一个大范围映射到一个小范围。把大范围映射到一个小范围的目的往往是为了节省空间，使得数据容易保存。

一般的说，Hash函数可以划分为如下几类：
加法hash 位运算hash 乘法hash 除法hash 查表hash 混合hash 数组hash

## 加法hash
所谓的加法Hash就是把输入元素一个一个的加起来构成最后的结果。

    static int additiveHash(String key, int prime)
     {
      int hash, i;
      for (hash = key.length(), i = 0; i < key.length(); i++)
       hash += key.charAt(i);
      return (hash % prime);
     }
## 位运算hash
通过利用各种位运算（常见的是移位和异或）来充分的混合输入元素。

    static int rotatingHash(String key, int prime)
     {
       int hash, i;
       for (hash=key.length(), i=0; i<key.length(); ++i)
         hash += (hash<<4)^(hash>>28)^key.charAt(i);
       return (hash % prime);
     }

## 乘法hash
这种类型的Hash函数利用了乘法的不相关性
static int bernstein(String key)

     {
       int hash = 0;
       int i;
       for (i=0; i<key.length(); ++i) hash = 33*hash + key.charAt(i);
       return hash;
     }
字符串的Hash。最简单可以使用基本的乘法Hash，当乘数为33时，对于英文单词有很好的散列效果（小于6个的小写形式可以保证没有冲突）。
## 除法Hash
除法和乘法一样，同样具有表面上看起来的不相关性。不过，因为除法太慢，这种方式几乎找不到真正的应用。需要注意的是，我们在前面看到的hash的 结果除以一个prime的目的只是为了保证结果的范围。如果你不需要它限制一个范围的话，可以使用如下的代码替代”hash%prime”： hash = hash ^ (hash>>10) ^ (hash>>20)。

## 查表Hash
查表Hash最有名的例子莫过于CRC系列算法。虽然CRC系列算法本身并不是查表，但是，查表是它的一种最快的实现方式。查表Hash中有名的例子有：Universal Hashing和Zobrist Hashing。他们的表格都是随机生成的。

## 混合Hash
混合Hash算法利用了以上各种方式。各种常见的Hash算法，比如MD5、Tiger都属于这个范围。它们一般很少在面向查找的Hash函数里面使用。

## 数组hash

    inline int hashcode(const int *v)
    {
     int s = 0;
     for(int i=0; i<k; i++)
        s=((s<<2)+(v[i]>>4))^(v[i]<<10);
     s = s % M;
     s = s < 0 ? s + M : s;
     return s;
    }
# Hash冲突解决方法
常见的哈希冲突解决办法有两种，开放地址法和链地址法。
## 一、开放地址法
开发地址法的做法是，当冲突发生时，使用某种探测算法在散列表中寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到。按照探测序列的方法，一般将开放地址法区分为**线性探查法、二次探查法、双重散列法**等。
### 线性探查
探查时从地址 d 开始，首先探查 T[d]，然后依次探查 T[d+1]，…，直到 T[m-1]，此后又循环到 T[0]，T[1]，…，直到探查到有空余的地址或者到 T[d-1]为止。
### 二次探查法
探查时从地址 d 开始，首先探查 T[d]，然后依次探查 T[d+di]，di 为增量序列1^2，-1^2，2^2，-2^2，……，q^2，-q^2 且q≤1/2 (m-1) ,直到探查到 有空余地址或者到 T[d-1]为止。
### 双重散列法
fi=(f(key)+i*g(key)) % m (i=1，2，……，m-1)
其中，f(key) 和 g(key) 是两个不同的哈希函数，m为哈希表的长度
步骤：
双哈希函数探测法，先用第一个函数 f(key) 对关键码计算哈希地址，一旦产生地址冲突，再用第二个函数 g(key) 确定移动的步长因子，最后通过步长因子序列由探测函数寻找空的哈希地址。

## 二、链地址法
![35574-31duq3ojk1a.png](http://www.sukidesu.top/usr/uploads/2020/03/1738397213.png)



