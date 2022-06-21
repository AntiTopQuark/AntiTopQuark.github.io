---
layout: post
cid: 67
title: [转]7天用Go从零实现RPC框架GeeRPC
slug: 67
date: 2021/04/13 20:28:00
updated: 2022/02/09 15:33:51
status: publish
author: AntiTopQuark
categories: 
  - 编程语言
tags: 
  - 接口
  - 实例
  - 程序
  - 方法
  - 调用
  - 语言
  - 状态
  - 进程
  - 数据
  - 原文
  - 笔者
  - 笔记
  - 记录
  - rpc
  - 框架
  - 机器
  - 协议
  - 客户端
  - 编码
  - option
  - header
  - 过程
  - 服务器
  - 参数
  - 程序员
  - 处理
  - 文本
  - 问题
customSummary: 
mathjax: auto
noThumbInfoStyle: default
outdatedNotice: no
pp_isEnabled: 0
pp_passwords: 
reprint: standard
thumb: 
thumbChoice: default
thumbDesc: 
thumbSmall: 
thumbStyle: default
---


  原文：https://geektutu.com/post/geerpc.html
这里是笔者用来进行记录的笔记。仅供学习和参考。
# RPC框架
## 1. 谈谈RPC
RPC(Remote Procedure Call，远程过程调用)是一种计算机通信协议，允许调用不同进程空间的程序。RPC 的客户端和服务器可以在一台机器上，也可以在不同的机器上。程序员使用时，就像调用本地程序一样，无需关注内部的实现细节。
不同的应用程序之间的通信方式有很多，比如浏览器和服务器之间广泛使用的基于 HTTP 协议的 Restful API。与 RPC 相比，Restful API 有相对统一的标准，因而更通用，兼容性更好，支持不同的语言。HTTP 协议是基于文本的，一般具备更好的可读性。但是缺点也很明显：
- Restful 接口需要额外的定义，无论是客户端还是服务端，都需要额外的代码来处理，而 RPC 调用则更接近于直接调用。
- 基于 HTTP 协议的 Restful 报文冗余，承载了过多的无效信息，而 RPC 通常使用自定义的协议格式，减少冗余报文。
- RPC 可以采用更高效的序列化协议，将文本转为二进制传输，获得更高的性能。
- 因为 RPC 的灵活性，所以更容易扩展和集成诸如注册中心、负载均衡等功能

## 2. RPC框架需要解决什么问题
我们可以想象下两台机器上，两个应用程序之间需要通信，那么首先，需要确定采用的传输协议是什么？如果这个两个应用程序位于不同的机器，那么一般会选择 TCP 协议或者 HTTP 协议；那如果两个应用程序位于相同的机器，也可以选择 Unix Socket 协议。传输协议确定之后，还需要确定报文的编码格式，比如采用最常用的 JSON 或者 XML，那如果报文比较大，还可能会选择 protobuf 等其他的编码方式，甚至编码之后，再进行压缩。接收端获取报文则需要相反的过程，先解压再解码。

解决了传输协议和报文编码的问题，接下来还需要解决一系列的可用性问题，例如，连接超时了怎么办？是否支持异步请求和并发？

如果服务端的实例很多，客户端并不关心这些实例的地址和部署位置，只关心自己能否获取到期待的结果，那就引出了注册中心(registry)和负载均衡(load balance)的问题。简单地说，即客户端和服务端互相不感知对方的存在，服务端启动时将自己注册到注册中心，客户端调用时，从注册中心获取到所有可用的实例，选择一个来调用。这样服务端和客户端只需要感知注册中心的存在就够了。注册中心通常还需要实现服务动态添加、删除，使用心跳确保服务处于可用状态等功能。

再进一步，假设服务端是不同的团队提供的，如果没有统一的 RPC 框架，各个团队的服务提供方就需要各自实现一套消息编解码、连接池、收发线程、超时处理等“业务之外”的重复技术劳动，造成整体的低效。因此，“业务之外”的这部分公共的能力，即是 RPC 框架所需要具备的能力。


# 第一天 服务端与消息编码
## 消息的序列化和反序列化
一个典型的 RPC 调用如下：
```go
err = client.Call("Arith.Multiply", args, &reply)
```
客户端发送的请求包括服务名 `Arith`，方法名 `Multiply`，参数 `args` 三个，服务端的响应包括错误 `error`，返回值 `reply` 2 个。我们将请求和响应中的参数和返回值抽象为 body，剩余的信息放在 header 中，那么就可以抽象出数据结构 Header：

GeeRPC 客户端固定采用 JSON 编码 Option，后续的 header 和 body 的编码方式由 Option 中的 CodeType 指定，服务端首先使用 JSON 解码 Option，然后通过 Option 得 CodeType 解码剩余的内容。即报文将以这样的形式发送：
```
| Option{MagicNumber: xxx, CodecType: xxx} | Header{ServiceMethod ...} | Body interface{} |
| <------      固定 JSON 编码      ------>  | <-------   编码方式由 CodeType 决定   ------->|
```
在一次连接中，Option 固定在报文的最开始，Header 和 Body 可以有多个，即报文可能是这样的。

```
| Option | Header1 | Body1 | Header2 | Body2 | ...

```