---
layout: post
title: "Angular HttpClient基本概念"
date: 2018-12-04
author: "WangX"
catalog: true
header-style: text
tags:
    - Angular
    - HttpClient
---

>现在浏览器支持使用两种不同的API发起HTTP请求：XMLHttpRequest接口和fetch()APT。@angular/common/http中的HttpClient类为Angular提供了一个简化的API来实现HTTP客户端功能。它基于XMLHttpRequest接口。 HttpClient 带来的其它优点包括：可测试性、强类型的请求和响应对象、发起请求与接收响应时的拦截器支持，以及更好的、基于可观察（Observable）对象的 API 以及流式错误处理机制。

* 把数据展现逻辑从数据访问逻辑中拆分出去，把数据访问逻辑包装进一个单独的服务中，并且在组件中把数据访问逻辑委托给这个服务，是一种常用的最佳实践。
* 要使用HttpClient，先要导入Angular的HttpClientModule,一般在根模块AppModule就导入它。之后就可以把HttpClient注入到数据访问的服务中。

## Get方法获取JSON数据
数据访问服务中代码：
```TypeScript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class ConfigService {
    //注入HttpClient，命名为http
  constructor(private http: HttpClient) { }
  configUrl = 'xxxx';

   //直接调用HttpClient的get方法，参数为url，返回一个Observable对象。
  getConfig() {
    return this.http.get(this.configUrl);
  }
}
```
数据展现逻辑中(组件中)代码：
```TypeScript
showConfig() {
  this.configService.getConfig()
    .subscribe((data: Config) => this.config = {
        heroesUrl: data['heroesUrl'],
        textfile:  data['textfile']
    });
}
```
组件中subscribe ConfigService中的getConfig()方法，回调函数把这些数据字段复制到组件的config对象中，config对象在模板中绑定，以供显示。