---
layout: post
title: "ES6中的解构赋值"
date: 2019-01-30 
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
    - ES6
---

## 解构赋值
>解构赋值语法是一个 Javascript 表达式，这使得可以将值从数组或属性从对象提取到不同的变量中。

#### 数组的解构赋值
* 从数组中提取值，按照对应位置，对变量赋值。       
```javascript
let [a,b,c] = [1,2,3];
//等价于
let a = 1;
let b = 2;
let c = 3;
```
