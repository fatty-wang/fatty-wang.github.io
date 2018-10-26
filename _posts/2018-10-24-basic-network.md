---
layout: post
title: "网络基础"
date: 2018-10-24 
author: "WangX"
catalog: true
header-style: text
tags:
    - network
    - linux
    - basic
---

## OSI七层协议
>OSI: Open System Interconnection


!["OSI七层模型"](/img/network/OSI.png "OSI七层模型")
上图中的OSI
1. 物理层：只能传输0和1这种比特信息，必须定义所使用的传输设备的信号，同时必须了解数据帧转成比特流的编码方式
2. 数据链路层：该层介于物理层和网络层之间，比较特殊。向下与物理层交互，偏硬，主要负责的是MAC(Media Access Control),这个数据包称为MAC数据帧，
MAC是网络接口设备所能处理的主要数据包裹，也是最终被物理层编码成比特流的数据。向上与网络层交互，偏软。主要指逻辑链接层(Logical Link Control,LLC)所控制，
主要在多任务处理来自上层的数据包数据(packer)并转换成MAC的格式，负责的工作包括信息交互，流量控制，失误问题的处理。
3. 网络层：IP(Internet Protocol)在这一层定义。同时也定义出计算机之间的连接建立，终止与维持等，数据包的传输路径选择等。最重要就是IP和路由(route)的概念。
4. 传输层：


!["OSI七层模型"](/img/network/TCP-IP.png "TCP/IP协议")