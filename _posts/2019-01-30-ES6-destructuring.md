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
* 只要等号两边的模式相同，左边的变量就会被赋予相应的值。
* 只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。
* 如果解构成功，变量的值就等于undefined。            
```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```   
* 解构赋值可以指定默认值：     
```javascript
let [foo = true] = [];
foo // true
let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```   
* ES6内部使用严格相等===，判断一个位置是否有值。即只有一个数组成员严格等于undefined，默认值才会生效。
* 如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。
* 默认值可以引用解构赋值的其他变量，但该变量必须已经声明。       
```javascript
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError: y is not defined 
```
#### 对象的解构赋值
* 数组的元素是按次序排列的，变量的取值有它的位置决定。对象的属性没有次序，**变量必须与属性同名**，才能取到正确的值。如果没有对应的同名属性，将赋予undefined。        
```javascript
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```
* 如果变量名与属性名不一致：         
```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```    
* 对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。下面代码中foo是匹配的模式，baz才是要赋值的变量。         
```javascript
let { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
foo // error: foo is not defined
```
* 对象解构的默认值生效的条件是，对象的属性值严格等于undefined。      
```javascript
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};
x // null
```
* 如果解构失败，变量的值等于undefined。

* 字符串可以解构赋值，字符串被转成一个类似数组的对象。
* 函数的参数也可以使用解构赋值。

## 应用
* 交换变量的值     
```javascript
let x = 1;
let y = 2;
[x,y] = [y,x];
```

* 从函数返回多个值
``` javascript
function f() {
    return [1,2,3];
}
let [a,b,c] = f();

function obj() {
    return {
        foo:1,
        bar:2
    };
}
let {foo,bar} = obj();
```
* 函数参数的定义    
```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

* 提取JSON    
```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};
let { id, status, data: number } = jsonData;
```

* 函数参数的默认值    
```javascript   
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

* 遍历Map结构     
```javascript
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```    
如果只选获取key,或者value
```javascript
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

* 输入模块的指定方法    
```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```