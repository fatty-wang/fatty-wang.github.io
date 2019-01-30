---
layout: post
title: "ES6中的let和const"
date: 2019-01-30 
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
    - ES6
---

## let
#### 块级作用域
* let 语句声明一个**块级作用域**(let命令所在的代码块内有效)的本地变量，并且可选的将其初始化为一个值。
* for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。      
```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
```  
上述代码中，函数内部的变量i和循环变量i不在同一个作用域，输出3次'abc'。
#### 不存在变量提升
* let命令不存在变量提升，let声明的变量一定要在声明后使用，否则报错。
#### 不可以重复声明
* let不允许在相同的作用域内，重复声明同一个变量。因此在函数内部不能直接重新声明函数。      
 
```javascript
function func(arg){
    let arg; //error
}

function func(arg) {
    {
        let arg;   //ok
    }
}
```
#### Temporal Dead Zone
* ES6中规定，区块中若存在let和const命令，则该区块在一开始对let和const命令声明的变量就形成了封闭作用域，若在声明前使用这些变量，就会报错。在代码块内，从一开始到let或const命令声明之前的这段区域称为"temporal dead zone"。      
```javascript
var v = 'a';
if (true) {
    v = 'b';   //ReferenceError
    let v;
}
```         
* 只要块级作用域内存在let命令，它所声明的变量就binding这个区域，不再受外部的影响。上述代码中，if语句的块级作用域内，let命令声明了变量v，那么v被绑定到这个区域，不受外面的全局变量v影响，所以在块级作用域内，仍然不可以在let声明之前，使用它。

## ES6中的块级作用域
* ES5只有全局作用域和函数作用域。 let为JavaScript增加了块级作用域。   
```javascript
function f() {
    let n = 5;
    {
        let n = 10;
    }
    console.log(n);
}
```   
* 上面代码中存在两个作用域，外层作用域不受内层作用域的影响。所以最终输出为10.
* ES6允许块级作用域任意嵌套。
1. **外层作用域无法读取内层作用域的变量**
2. **内层作用域可以声明和外层作用域相同的变量**

#### ES6块级作用域中的函数声明
* ES6 引入了块级作用域，明确允许在块级作用域之中声明函数（在使用大括号的情况下）。ES6 规定，块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。
* 但是为了减轻因此产生的“不兼容问题”，浏览器可以不遵守上述规定，有自己的行为方式：
1. 允许在块级作用域内声明函数。
2. 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
3. 函数声明还会提升到所在的块级作用域的头部。

* 考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。如下：  
```javascript
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```

## const

* const命令声明一个只读的常量。const声明变量时必须初始化，初始化后不能改变。
* const命令和let命令一样：作用域相同、变量不提示、存在temporal dead zone、不可重复声明。
* const保证的是**变量指向的那个内存地址所保存的数据不得改动**。
1. 对于数值、字符串、布尔值，值就保存在变量指向的那个内存地址，因此等同于常量。
2. 对于复合类类型数据，变量指向的内存地址保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的。

## ES6中的声明变量
* ES6中共有6中声明变量的方法：
1. ES5中的var命令和function命令。
2. ES6中添加的let命令、const命令、import命令和class命令。

## 顶层对象
* 顶层对象在浏览器中指的是window对象，在Node指的是global对象。ES5中，顶层对象的属性与全局变量是等价的。
* ES6中为了改变顶层对象与全局变量挂钩：
1. 为兼容性，var和function声明的全局变量，依旧是顶层对象的属性
2. let命令、const命令、class命令声明的全局变量，**不属于顶层对象的属性**。
3. 从ES6开始，全局变量将逐步与顶层对象的属性脱钩。
* ES5中的顶层对象，不通环境中也不一样：
1. 浏览器中的顶层对象是window
2. 浏览器和Web Worker里面，self也指向顶层对象
3. Node中顶层对象是global。