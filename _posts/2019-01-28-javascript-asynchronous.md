---
layout: post
title: "Javascript中的异步"
date: 2019-01-19 
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
    - 异步
---

## 异步概述

#### 单线程模型
* JavaScript只在一个线程上运行，即同时只能执行一个任务。为了降低浏览器的复杂性。
* JavaScript引擎有多个线程，单个脚本只能再一个线程（主线程）上运行，其他线程在后台配合。
* JavaScript 内部采用的“事件循环”机制（Event Loop）。
* 为了利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不得操作 DOM。这个新标准并没有改变 JavaScript 单线程的本质。
#### 同步任务和异步任务
* 同步任务(synchronous)是那些没有被引擎挂起、在主线程上排队执行的任务。只有前一个任务执行完毕，后一个任务才会执行。
* 异步任务(asynchronous)被放入任务队列中。当引擎认为某个异步任务可以执行了，该任务以回调函数的形式进入主线程执行。
#### 任务队列和事件循环
* JavaScript运行时，除去主线程，引擎还提供一个任务队列，里面是各种需要当前程序处理的异步任务。实际上，根据异步任务的类型，存在多个任务队列。
* 主线程执行所有的同步任务，同步任务执行完就去查看任务队列里面的异步任务。如果满足条件，异步任务就重新进入主线程开始执行。一旦任务队列清空，程序就结束执行。
* 异步任务的写法通常是回调函数。异步任务重新进入主线程就会执行相应的回到函数。如果一个异步任务没有回调函数，就不会进入任务队列。
* 只要同步任务执行完了，引擎就会不停地检测那些挂起来的异步任务，是不是可以进入主线程。这种循环检查的机制，叫做事件循环（Event Loop）。

## 异步操作的几种模式

#### 回调函数
* 回调函数的优点是简单、容易理解和实现，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（coupling），使得程序结构混乱、流程难以追踪（尤其是多个回调函数嵌套的情况），而且每个任务只能指定一个回调函数。
#### 事件监听
* 事件监听优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以“去耦合”（decoupling），有利于实现模块化。缺点是整个程序都要变成**事件驱动型**，运行流程会变得很不清晰。阅读代码的时候，很难看出主流程。
#### 发布/订阅 （观察者模式）
* 事件完全可以理解成“信号”，如果存在一个“信号中心”，某个任务执行完成，就向信号中心publish一个信号，其他任务可以向信号中心subscribe这个信号，从而知道时候自己可以开始执行。这种模式称为发布/订阅模式，或观察者模式。
* 这种方法的性质与“事件监听”类似，但是明显优于后者。因为可以通过查看“消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

## 定时器
#### setTimeout()
* setTimeout函数用来指定某个函数或某段代码，在多少毫秒之后执行。它返回一个整数，表示定时器的编号，以后可以用来取消这个定时器。     
```javascript
var timerId = setTimeout(func|code,delay);
```
setTimeout接收两个参数，第一个参数是将要推迟执行的函数名或一段代码(字符串形式)，第二个参数是推迟执行的毫秒数。
* 除了上面两个参数，setTimeout允许更多的参数，它们将以此传入推迟执行的函数也就是回调函数中。
* 如果回调函数时对象的方法，那么setTimeout是的方法内部的this关键字指向全局环境，而不是定义时所在的那个对象。       
```javascript
var x = 1;
var obj = {
    x:2,
    y: function () {
        console.log(this.x);
    }
};

setTimeout(obj.y,1000);
```
上面代码的输出为1，而不是2。因为当obj.y在1000毫秒后运行时this所指向的已经不是obj了，而是全局环境。可以将obj.y放入匿名函数中，obj.y将在obj的作用域执行。或者使用bind方法，将obj.y绑定到obj上面。
```javascript
setTimeout(function () {
    obj.y();
},1000);

setTimeout(obj.y.bind(obj),1000);
```
#### setInterval()
* 与setTimeout完全一致，区别仅仅在于setInterval指定某个任务每隔一段时间就执行一次，也就是无限次的定时执行。
* setInterval指定的是“开始执行”之间的间隔，并不考虑每次任务执行本身所消耗的时间。若任务执行消耗的时间超过指定的时间间隔，则会立即执行下一次。
* 为了确保两次执行之间有固定的间隔，可以不用setInterval，而是每次执行结束后，使用setTimeout指定下一次执行的具体时间。

#### clearTimeout()，clearInterval()
* setTimeout和setInterval函数，都返回一个整数值，表示计数器编号。将该整数传入clearTimeout和clearInterval函数，就可以取消对应的定时器。

#### debounce函数
* debounce(防抖动)，不希望回调函数被频繁调用。   
```javascript
function debounce(fn,delay){
    var timer = null;
    return function() {
        var context = this;
        var args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function () {
            fn.apply(context,args);
        },delay);
    };
}
$('textarea').on('keydown',debounce(ajaxAction,2500));
```
上面代码中，只要在2500毫秒之内，用户再次击键，就会取消上一次的定时器，然后再新建一个定时器。这样就保证了回调函数之间的调用间隔，至少是2500毫秒。

#### 运行机制
* setTimeout和setInterval的运行机制，是将指定的代码移出本轮事件循环，等到下一轮事件循环，再检查是否到了指定时间。如果到了，就执行对应的代码；如果不到，就继续等待。
* setTimeout和setInterval指定的回调函数，必须等到本轮事件循环的所有同步任务都执行完，才会开始执行。由于前面的任务到底需要多少时间执行完，是不确定的，所以没有办法保证，setTimeout和setInterval指定的任务，一定会按照预定时间执行。
* setTimeout(f, 0)会在下一轮事件循环一开始就执行。