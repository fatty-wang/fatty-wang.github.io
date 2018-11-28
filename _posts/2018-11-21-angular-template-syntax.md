---
layout: post
title: "Angular 模板语法"
date: 2018-11-21 
author: "WangX"
catalog: true
header-style: text
tags:
    - Angular
---

>在 Angular 中，组件扮演着控制器或视图模型的角色，模板则扮演视图的角色。

>HTMl是Angular模板的语言。\<script>元素被忽略，以阻止脚本注入攻击的风险。\<html>、\<body>和\<base>元素在模板中没有多大意义。

## 插值表达式 {{}}

* 双花括号括起来的插值表达式，把计算后的字符串插入到HTML元素标签内的文本或对标签的属性进行赋值。
* Angular 对所有双花括号中的表达式求值，把求值的结果转换成字符串，并把它们跟相邻的字符串字面量连接起来。最后，把这个组合出来的插值结果赋给元素或指令的属性。
* 插值表达式是一个特殊的语法，Angular把它转换成了属性绑定。

## 模板表达式

* Angular执行模板表达式，产生一个值并赋值给HTML元素、组件或指令的属性。
* 模板表达式语言是JavaScript的子集，并补充了几个用于特定场景的特殊操作符。

#### 表达式上下文

* 表达式中的上下文变量是由模板变量、指令的上下文变量和组件的成员叠加而成的。如果你要引用的变量名存在于多个命名空间中，则模板变量是最优先的，其次是指令的上下文变量，最后是组件的成员。
* 模板表达式只能引用表达式上下文中的成员， 不能引用全局命名空间中的任何到东西，比如window或document。

#### 表达式使用规范

* 模板表达式除了目标属性的值以外，不应该改变应用的任何状态。这是Angular ***单向数据流***策略的基础。不用担心读取组件值可以能改变另外的显示值。
* 逻辑简单，执行迅速。Angular会在每个变更检测周期后执行模板表达式。应该在组件中实现应用和业务逻辑。
* 幂等性，调用多次返回的完全相同的东西。

## 模板语句

* 模板语句用来响应由绑定目标(如HTML元素、组件或指令)触发的事件。
```html
(event)="statement"
```
出现在"=号"右侧的引号中。

#### 模板语句上下文

* 典型的语句上下文就是当前组件的实例。语句上下文可以引用模板自身上下文中的属性。如模板的$event对象，模板输入变量(let hero)和模板引用变量(#heroForm)。
* 模板上下文中的变量名的优先级高于组件上下文中的变量名。
* 避免写复杂的模板语句，一般是函数调用或者属性赋值。

## 绑定语法

> 绑定的类型可以根据数据流的方向分为三类：从数据源到视图、从视图到数据源以及双向的从视图到数据源再到视图。

#### HTML attribute 与 Dom property的对比
* attribute是由HTML定义的。property是由DOM(Document Object Model)定义的。
* attribute初始化DOM property，property的值可以改变，attribute的值不能改变。property是当前值。
* 模板绑定通过property和事件来工作，而不是attribute，attribute 唯一的作用是用来初始化元素和指令的状态。

|数据方向|类型|
|:-:|:-:|
|从数据源到视图|插值表达式、属性(property)、Attribute、CSS类、样式|
|从视图到数据源|事件|
|双向|双向|

#### 绑定目标

>数据绑定的目标是DOM中的内容，根据绑定类型，绑定目标可以是(元素、组件、指令的)property,(元素、组件、指令的)事件、或极少情况下的attribute名。

|绑定类型|目标|例子|
|:-:|:-:|:-:|
|属性|元素的property<br>组件的property<br>指令的property|\<img [src]="heroImageUrl"> <br> \<app-hero-detail [hero]="currentHero">\</app-hero-detail> <br> \<div [ngClass]="{'special':isSpecial}">\</div>|
|事件|元素的事件<br>组件的事件<br>指令的事件|\<button (click)="onSave()">Save\</button> <br>\<app-hero (deleteRequest)="deleteHero()">\</app-hero><br>\<div (myClick)="clicked=$event" clickable>click me\</div>|
|双向|事件与property|\<input [(ngModule)]="name">|
|Attribute|attribute(例外情况)|\<button [attr.aria-label]="help">help\</button>|
|CSS 类|class property|\<div [class.special]="isSpecial">Special\</div>|
|样式|style property|\<button [style.color]="isSpecial ? 'red' : 'green'">|

* Property binding([property])：把视图元素的property设置为模板表达式。最常用的property binding是把元素属性设置为组件属性的值。如下：
```html
<img [src]="heroImgUrl">
```
上面还可以使用bind-前缀的规范形式：
```html
<img bind-src="heroImageUrl">
```
另外一个例子是设置自定义组件的模型属性（这是父子组件之间通信的重要方式）：
```html
<app-hero-detail [hero]="currentHero"></app-hero-detail>
```
#### 单向输入
*  property binding 被称为单向数据绑定，值的流动是单向的从组件的数据属性流动到目标元素的属性。

* 元素属性可能是最常见的绑定目标，但Angular会首先判断这个名字是否是某个已知指令的属性名。

* []方括号告诉Angular要计算目标表达式。如果没有方括号，Angular会把目标表达式当做字符串常量来初始化目标常量。

#### 属性绑定 OR 插值表达式

* 在多数情况下，插值表达式是更方便的备选项。实际上，在渲染视图之前，Angular把这些插值表达式翻译成相应的属性绑定。当时渲染的数据类型是字符串时，插值表达式的可读性更强。但数据类型不是字符串时，就必须使用属性绑定了。
属性绑定：
```html
<img [src]="heroImgUrl">
```
插值表达式：
```html
<img src="{{heroImageUrl}}">
```
#### 内容安全
* 不管是插值表达式还是属性绑定，都不会允许带有 script 标签的 HTML 泄漏到浏览器中。

#### attribute绑定
>当元素没有property可以绑定的时候，就必须使用attribute绑定。
* 插值表达式和属性绑定只能设置property，不能设置attribute。
* 考虑 ARIA， SVG 和 table 中的 colspan/rowspan 等 attribute。 它们是纯粹的 attribute，没有对应的属性可供绑定。
* attribute绑定的语法和property绑定类似，但方括号中的部分不是元素的property名称，而是有attr前缀，一个点和attribute名字组成。并通过字符串的表达式来设置attribute的值。如下：
```html
<tr><td [attr.colspan]="1 + 1">One-Two</td></tr>
```
#### CSS类绑定

