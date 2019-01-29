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

## Promise
> Promise是异步编程的一种方案，优于回调函数和事件驱动，最早由社区提出和实现，ES6将其写进了语言标准，统一了用法，原生提供了Promise对象。

* Promise对象中保存着某个未来才会结束的事件(通常是一个异步操作)的结果。Promise对象的两个特点：
1. Promise对象代表的异步操作有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）。只有异步操作的结果可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
2. 一旦状态改变，就不会再变，**任何时候都可以得到这个结果**。Promise对象的状态改变，只有两种情况：从pending变为fulfilled和从pending到变为rejected。只要这两种情况发生，就不会再变，称为resolved（已定型）。
* **如果状态已经发送改变，再对Promise添加回调函数，也会立即得到这个结果，这与Event完全不同，如果错过Event的发生，再去监听是不会得到结果**。
* Promise缺点：
1. Promise无法取消，一旦新建就会立即执行，无法中途取消。
2. 如果不设置回调函数，Promise内部抛出错误、不会反应到外部。
3. 当处于Pending状态是，无法得知目前是刚刚开始，还是即将完成。

#### 基本用法
* ES6中，Promise对象是一个构造函数，用来生成Promise实例。Promise构造函数接收一个**函数**作为参数，该函数的**两个参数**分别是resolve和reject这个**两个函数**(由JavaScript引擎提供)。
1. resolve函数将Promise对象状态从pending变为fulfilled，在异步操作成功时调用，并**异步操作的结果，作为参数传递出去**;
2. reject函数将Promise对象状态从pending到变为rejected，在异步操作失败时调用,并将**异步操作报出的错误，作为参数传递出去**。     
```javascript
const promise = new Promise(function(resolve, reject) {
  // ... some code
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```    

* Promise实例生成后，可以用then方法分别指定resolved状态和rejected状态的回调函数。then方法接收两个回调函数作为参数。第一个回调函数为状态变为fulfilled时调用，第二回调函数为状态变为rejected时调用。其中第二个回调函数是**可选的**，这两个回调函数都接受**Promise对象传出的值作为参数**。      
```javascript
promise.then(function(value){
    //success
},fucntion(error){
    //failure
});
```   
下面为一个Promise对象例子：     
```javascript
function timeout(ms){
    return new Promise((resolve,reject)=>{
        setTimeout(resolve,ms,'done');
    });
}
timeout(1000).then((value)=>{
    console.log(value);
});
```     
上述例子中:timeout函数接收一个毫秒数作为参数，返回一个Promise实例，Promise实例接收一个函数为参数。该函数中定义了resolve函数异步触发，并且给resolve函数传递参数'done'。**如果调用resolve函数和reject函数时带有参数，那么它们的参数会被传递给回调函数**。then方法中只有一个参数，作为resolve函数触发后的回调函数，该回调函数打印'done'。

* Promise新建后会立即执行。     

```javascript
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');
```    
上述代码中，Promise新建后立即执行，最先输出'Promise'，**then方法中指定的回调函数，将在当前脚本所有同步任务执行完再执行**，所以第二输出的是'Hi!'，最后输出的是'resolved'。

*传递给**resolve函数**的参数除了正常值外，还可以是另外一个Promise实例。**这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。** 如下：    
 
```javascript
const p1 = new Promise(function (resolve, reject) {
  // ...
});

const p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```     
* resolve或reject的调用并不会终结Promise的参数函数的执行。     
```javascript
new Promise((resolve, reject) => {
  resolve(1);
  // return resolve(1);
  console.log(2);
}).then(r => {
  console.log(r);
});
```     
调用resolve(1)后，console.log(2)还是会执行，并且首先打印出来。这是因为立即resolved的Promise是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。最好在前面加上return语句。

#### Promise.prototype.then(onFulfilled, onRejected)

* then方法时定义在Promise.prototype上的，then方法也返回一个**新的**Promise实例，因此可以采用**链式写法**，也就是then方法后面再调用另一个then方法。下面代码中，第一个回调函数完成后，会将返回结果作为参数，传入第二个回调函数。        
```javascript
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
});
```     
#### Promise.prototype.catch(onRejected)

* Promise.prototype.catch方法是.then(null,rejection)或.then(undefined,rejection)的别名，用于指定发生错误时的回调函数。     
```javascript
getJSON('/posts.json').then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```    
上述代码中getJSON返回一个Promise对象。如果对象状态从pending变为fulfilled则会调用then方法中指定的回调函数；如果变为rejected就会调用catch方法指定的回调函数。**then方法指定的回调函数在运行中***抛出***错误，也会被catch方法捕获**。

* reject方法的作用等同于抛出错误！
* Promise 对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。      
```javascript
getJSON('/post/1.json').then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```   
上面代码中共有三个Promise对象，一个由getJSON生成，两个有then生成，任何一处抛出错误或reject都会由最后一个catch捕获。般来说，不要在then方法里面定义 Reject 状态的回调函数（即then的第二个参数），总是使用catch方法。       
```javascript
promise
  .then(function(data) { //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```     
上面写法中的catch既能捕捉到promise中的rejected，也能捕捉到then中执行中的错误。catch方法返回的还是一个 Promise 对象。

#### Promise.prototype.finally(onFinally)     
* finally() 方法返回一个Promise，在promise执行结束时，无论结果是fulfilled或者是rejected，在执行then()和catch()后，都会执行finally指定的回调函数。这为指定执行完promise后，无论结果是fulfilled还是rejected都需要执行的代码提供了一种方式，避免同样的语句需要在then()和catch()中各写一次的情况。
* onFinally为Promise 状态改变后执行的回调函数，不接受任何参数，这意味着没有办法知道，前面的 Promise 状态到底是fulfilled还是rejected。
* finally本质上是then方法的特例。

#### Promise.all(iterable)
* Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。在 iterable 参数内的多个实例全部变成fulfilled，新实例的状态才会变成fulfilled。有一个被rejected，新实例的状态就是rejected，此时第一个被reject的实例返回值，会传递出来。

#### Promise.race(iterable)
* Promise.race(iterable) 方法返回一个 promise，一旦迭代器中的某个promise解决或拒绝，返回的 promise就会解决或拒绝。

#### Promise.resolve(value)
* Promise.resolve(value)方法返回一个以给定值解析后的Promise 对象。但如果这个值是个thenable（即带有then方法），返回的promise会“跟随”这个thenable的对象，采用它的最终状态；如果传入的value本身就是promise对象，则该对象作为Promise.resolve方法的返回值返回；否则以该值为成功状态返回promise对象。

#### Promise.reject(reason)
* Promise.reject(reason)方法返回一个带有拒绝原因reason参数的Promise对象。

