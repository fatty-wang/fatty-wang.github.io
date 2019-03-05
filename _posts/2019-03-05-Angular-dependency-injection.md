---
layout: post
title: "Angular中的依赖注入 "
date: 2019-03-05
author: "WangX"
catalog: true
header-style: text
tags:
    - Angular
    - DI
---

## 依赖注入相关概念

>    In software engineering, dependency injection is a technique whereby one object (or static method) supplies the dependencies of another object. A dependency is an object that can be used (a service). An injection is the passing of a dependency to a dependent object (a client) that would use it. The service is made part of the client's state.Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern.                      ----wikipedia

* 依赖注入意在解耦：使得客户端的代码不必跟着它依赖对象的改变而改变。
* 依赖注入是控制反转（概念更加宽泛的技术）的一种形式。客户端将提供其依赖的职责委托给外部代码（注入器）。客户端不允许直接调用注入代码，是注入代码来构建服务，并注入给客户端。也就是说，客户端代码不需要了解注入代码是如何构建服务，甚至不知道哪个真实的服务注入代码正在使用。客户端只需要知道它依赖的那些服务的固有接口，从而调用。这样就把使用和构造的责任分离开来。
* 依赖注入涉及4个角色：
1. 被使用的**服务**对象
2. 依赖的服务的**客户端**对象
3. 定义客户端怎样使用服务的**接口**
4. 构造服务并把服务注入客户端的**注入器**


## Angular中的依赖注入

#### 概述

!["DI框架构成"](/img/angular2/di-in-angular2-5.svg "DI框架构成")
