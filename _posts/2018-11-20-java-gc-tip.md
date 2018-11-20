---
layout: post
title: "Java GC 日志格式"
date: 2018-11-20 
author: "WangX"
catalog: true
header-style: text
tags:
    - JAVA
    - JAVA GC
    - basic
---

## GC日志格式

>每个收集器的日志格式都可以不一样，但都保持一定的共性

!["JAVA GC日志样例"](/img/javagc/javagctip.png "JAVA GC日志样例")

* 最前面的数字67009.507和67348.239代表了GC发生的时间，是指从JAVA虚机机启动以来经过的秒数。
* "[Full GC"和 "[GC"说明了这次垃圾收集的停顿类型。
* "[PSYoungGen"、"[PSOldGen"、"[PSPermGen"表示GC发生的区域，这个显示的区域名称与使用的CG收集器是密切相关的。这里使用的是Parallel Scavenge收集器。
* **方括号内部的** "277696K->8624K(313600K)" 对应着：“GC前该内存区域已使用容量->GC后该内存区域已使用容量（该内存区域总容量）”
* **方括号之外的** "418801K->149730K(1012672K)" 对应着：“GC前Java堆已使用容量->GC后Java堆已使用容量（Java堆总容量）”
* 再往回 "0.0131290 secs" 表示该内存区域GC所占用的时间。
* "[Times: user=0.04 sys=0.00, real=0.02 secs]" 对应着：用户态消耗的CPU时间，内核态消耗的CPU时间和操作从开始到结束所经过的墙钟时间（Wall Clock Time）。
WCT包括各种非运算时间，若I/O等待，而CPU时间不包括。