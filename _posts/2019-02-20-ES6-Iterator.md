---
layout: post
title: "ES6中的Iterator"
date: 2019-02-20
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
    - ES6
---

## 概念
* Iterator（迭代器）是为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作。
* Iterator的三个作用：
1. 为各种数据结构，提供一个统一的、简便的访问接口
2. 使得数据结构的成员能够按某种次序排列
3. S6 创造了一种新的遍历命令for...of循环，Iterator 接口主要供for...of消费
* Iterator的遍历过程：
1. 创建一个指针对象，指向当前数据结构的起始位置。迭代器对象的本质上，就是一个指针对象。
2. 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
3. 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
4. 不断调用指针对象的next方法，直到它指向数据结构的结束位置。
* 每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。
* Iterator 只是把接口规格加到数据结构之上，所以，迭代器与它所遍历的那个数据结构，实际上是分开的，完全可以写出没有对应数据结构的迭代器对象，或者说用迭代器对象模拟出数据结构。

```javascript
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

//makeIterator函数返回一个Iterator
function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}

//无限运行的遍历器对象

it.next().value // 0
it.next().value // 1
it.next().value // 2
// ...

function idMaker() {
  var index = 0;

  return {
    next: function() {
      return {value: index++, done: false};
    }
  };
```
* 对于遍历器对象来说，done: false和value: undefined属性都是可以省略的。
* 如果使用 TypeScript 的写法，遍历器接口（Iterable）、指针对象（Iterator）和next方法返回值的规格可以描述如下。    

```typescript
interface Iterable {
  [Symbol.iterator]() : Iterator,
}

interface Iterator {
  next(value?: any) : IterationResult,
}

interface IterationResult {
  value: any,
  done: boolean,
}
```
## 默认Iterator接口
* 当使用for...of循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。
* 一种数据结构只要部署了 Iterator 接口，我们就称这种数据结构是“可遍历的”（iterable）。
* ES6规定，默认的Iterator接口部署在数据结构的Symbol.iterator属性。只有一个数据结构具有Symbol.iterator属性，就可任务是iterable。Symbol.iterator属性本身是一个函数，就是当前数据结构默认的遍历器生成函数。执行这个函数，就会返回一个遍历器。 
* Symbol.iterator是一个表达式，返回Symbol对象的iterator属性。这是ES6内置的类型为Symbol的特殊值。
* 下面代码中的obj具有Symbol.iterator属性，是iterable的。Symbol.iterator返回一个迭代器对象。迭代器对象的根本特征就是具有next方法，每次调用next方法，都会返回一个代表当前成员的信息对象，具有value和done两个属性。

```javascript
const obj = {
    [Symbol.iterator] : function () {
        return {
            next: function () {
                return {
                    value:1,
                    done:true
                };
            }
        };
    }
};
```
* 原生具备Iterator接口的数据结构有：Array、Map、Set、String、TypedArray、函数的arguments对象和NodeList对象。
* 一个对象(Object)如果要具备可被for...of循环调用的 Iterator 接口，就必须在Symbol.iterator的属性上部署遍历器生成方法（原型链上的对象具有该方法也可）。
* 对于类似数组的对象（存在**数值键名**和**length属性**），部署 Iterator 接口，有一个简便方法，就是Symbol.iterator方法直接引用数组的 Iterator 接口。   

```javascript
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // 'a', 'b', 'c'
}
```

## 调用Iterator接口的场合
* 解构赋值，对数组和 Set 结构进行解构赋值时，会默认调用Symbol.iterator方法。
* 扩展运算符（...）也会调用默认的 Iterator 接口。只要某个数据结构部署了 Iterator 接口，就可以对它使用扩展运算符，将其转为数组。

```javascript
let arr = [...iterable];
```
* yield*后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。
* 由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。

## 字符串的Iterator接口
* 字符串是一个类似数组的对象，也原生具有 Iterator 接口。可以覆盖原生的Symbol.iterator方法，达到修改遍历器行为的目的。

```javascript
var str = new String("hi");

[...str] // ["h", "i"]

str[Symbol.iterator] = function() {
  return {
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};

[...str] // ["bye"]
str // "hi"
```
## Iterator中的return(),throw()
* 在Iterator对象中，next方法是必选的。此外，还可以具有return方法和throw方法。

```javascript
function readLinesSync(file) {
  return {
    [Symbol.iterator]() {
      return {
        next() {
          return { done: false };
        },
        return() {
          file.close();
          return { done: true };
        }
      };
    },
  };
}

// 情况一 输出文件第一行后，就会执行return方法
for (let line of readLinesSync(fileName)) {
  console.log(line);
  break;
}

// 情况二 会在执行return方法后，再抛出错误
for (let line of readLinesSync(fileName)) {
  console.log(line);
  throw new Error();
}
```
**情况二 会在执行return方法后，再抛出错误**

* 如果for...of循环提前退出（通常是因为出错，或者有break语句），就会调用return方法。return方法必须返回一个对象，这是 Generator 规格决定的。

## for...of循环
* for...of循环内部调用的是数据结构的Symbol.iterator方法。
* 数组：

```javascript
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
}
```
* JavaScript 原有的for...in循环，只能获得对象的键名，不能直接获取键值。ES6 提供for...of循环，允许遍历获得键值。
* for...of循环调用遍历器接口，数组的遍历器接口**只返回具有数字索引**的属性。这一点跟for...in循环也不一样。下面代码中，for...of不会返回数组的foo属性。          

```javascript
let arr = [3, 5, 7];
arr.foo = 'hello';

for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}

for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
```
* Set和Map结构：

```javascript
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```
* 遍历的顺序是按照各个成员被**添加**进数据结构的**顺序**。
* Set 结构遍历时，返回的是一个值，而 Map 结构遍历时，返回的是一个数组，该数组的两个成员分别为当前 Map 成员的键名和键值。