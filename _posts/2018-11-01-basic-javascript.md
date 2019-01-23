---
layout: post
title: "JavaScript拾遗"
date: 2019-11-01 
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
---

## 导论

* JavaScript 的基本语法和对象体系，是模仿 Java 而设计的。但是，JavaScript 没有采用 Java 的静态类型。JavaScript 语言的函数是一种独立的数据类型，以及采用基于原型对象（prototype）的继承链。这是它与 Java 语法最大的两点区别。JavaScript 语法要比 Java 自由得多。

* ECMAScript 和 JavaScript 的关系是，前者是后者的规格，后者是前者的一种实现。在日常场合，这两个词是可以互换的。ECMAScript 只用来标准化 JavaScript 这种语言的基本语法结构，与部署环境相关的标准都由其他标准规定，比如 DOM 的标准就是由 W3C组织（World Wide Web Consortium）制定的。

* JavaScript 是一种动态类型语言，也就是说，变量的类型没有限制，变量可以随时更改类型。JavaScript 引擎的工作方式是，先解析代码，获取所有被声明的变量，然后再一行一行地运行。这造成的结果，就是所有的变量的声明语句，都会被提升到代码的头部，这就叫做变量提升（hoisting）。

## 语法

* 标签(label)
JavaScript 语言允许，语句的前面有标签（label），相当于定位符，用于跳转到程序的任意位置，标签的格式如下。
```javascript
label:
  语句
```
标签可以是任意的标识符，但不能是保留字，语句部分可以是任意语句。标签通常与break语句和continue语句配合使用，跳出特定的循环。
```javascript
top:
  for (var i = 0; i < 3; i++){
    for (var j = 0; j < 3; j++){
      if (i === 1 && j === 1) break top;
      console.log('i=' + i + ', j=' + j);
    }
  }
```
上面代码为一个双重循环区块，break命令后面加上了top标签（注意，top不用加引号），满足条件时，直接跳出双层循环。如果break语句后面不使用标签，则只能跳出内层循环，进入下一次的外层循环。
标签也可以用于跳出代码块:
```javascript
foo: {
  console.log(1);
  break foo;
  console.log('本行不会输出');
}
console.log(2);
```
上面代码执行到break foo，就会跳出区块。 
continue语句也可以与标签配合使用:
```javascript
top:
  for (var i = 0; i < 3; i++){
    for (var j = 0; j < 3; j++){
      if (i === 1 && j === 1) continue top;
      console.log('i=' + i + ', j=' + j);
    }
  }
```
ontinue命令后面有一个标签名，满足条件时，会跳过当前循环，**直接进入下一轮外层循环**。如果continue语句后面不使用标签，则只能进入下一轮的内层循环。

## 数据类型

* ES6之前 JavaScript有六种数据类型：number、string、boolean、undefined、null和object。ES6新增了Symbol类型的值。
* typeof运算符和返回一个值的数据类型。函数返回function。
* null是一个表示“空”的对象，转为数值时为0；undefined是表示一个“此处无定义”的原始值，转为数值时为NaN。


* 如果 JavaScript 预期某个位置应该是布尔值，会将该位置上现有的值自动转为布尔值。转换规则是除了下面六个值被转为false，其他值都视为true。
undefined
null
false
0
NaN
""或''（空字符串）

#### 数值
* JavaScript内部，所有数字都是以64位浮点数形式储存，1与1.0是**相同**的。
* 64位中，第1位：符号位。第2位到第12位：指数部分。第13位到64位：小数部分。
* NaN 表示 Not a Number，主要出现在将字符串解析成数字出错时。NaN属于number类型。NaN不等于任何值，包括它自身。
* Infinity表示无穷。
#### 字符串
* 字符串默认只能写在一行内，分成多行将会报错。如果长字符串必须分成多行，可以在每一行的尾部使用反斜杠。
```javascript
var longString = 'Long \
long \
long \
string';
```
* 字符串可以使用字符数组的方式来返回某个位置的字符。仅此而已，不可以改变字符串中的某个单个字符。
* length属性返回字符串的长度，该属性无法改变。
* JavaScript 使用 Unicode 字符集。JavaScript 引擎内部，所有字符都用 Unicode 表示。

#### 对象
* 对象就是一组 key-value 的集合，是一种无序的复合数据集合。
* 对象的所有key名都是字符串，ES6中的Symbol值也可以作为key，所以加不加引号都可以。
* 对象采用大括号表示，这导致了一个问题：如果行首是一个大括号，它到底是表达式还是语句？ JavaScript 引擎的做法是，如果遇到这种情况，无法确定是对象还是代码块，一律解释为代码块。
* 对象属性的读取和赋值，有两种方法，一种是使用点运算符，还有一种是使用方括号运算符。如果使用方括号运算符，键名必须放在引号里面。
* Object.keys(obj)可以查看obj的所有key。
* delete命令

