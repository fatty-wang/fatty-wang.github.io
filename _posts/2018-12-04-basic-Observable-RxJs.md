---
layout: post
title: "Observable与RxJs基本概念"
date: 2018-12-04
author: "WangX"
catalog: true
header-style: text
tags:
    - Angular
    - Observable
    - RxJs
---

## Observable(可观察对象)

* Observable支持在发布者和订阅者之间传递消息。
* Observable是声明式的，**在消费者订阅它之前，定义的用于发布值的函数并不会实际执行。**订阅之后，当这个函数执行完成或取消订阅时，订阅者就会收到通知。
* Observable可以发送多个任意类型的值：字面量、消息、事件。无论是同步发送还是异步发送，接收这些值的API是一样的。应用代码只管订阅并消费这些值，其他逻辑由Observable自己处理。

## 基本用法和词汇

* 发布者创建一个Observable实例，Obeservable实例中定义了一个**订阅者函数**，当有消费者调用subscribe()方法时，这个函数就会执行。**订阅者函数用于定义：如何获取或生成那些要发布的值或消息**。
* 订阅者调用Observable实例的subscribe()方法，并传入一个观察者(observer)。observer定义了收到的消息的处理器。subscribe()调用会返回一个Subscription对象，该对象具有一个unsubscribe()方法。调用改方法时，就会停止接收通知。

#### 定义观察者

用于接收可观察对象通知的处理器要实现Observer接口。这个对象定义了一些回调函数来处理可观察对象可能会发来的三种通知：
1. next。必须有。用来处理每个送达值。在开始执行后可能执行零次或多次。
2. error。可以有。用来处理错误通知。错误会中断这个可观察对象实例的执行过程。
3. complete.可以有。用来处理执行完毕（complete）通知。当执行完毕后，这些值就会继续传给下一个消息处理器。
观察者对象可以定义这三种处理器的任意组合。

#### 订阅
只有当有人订阅 Observable 的实例时，它才会开始发布值。订阅时要先调用该实例的subscribe()方法，并把一个观察者对象传给它，用来接收通知。
> of(...items) —— 返回一个 Observable 实例，它用同步的方式把参数中提供的这些值发送出来。    
> from(iterable) —— 把它的参数转换成一个 Observable 实例。 该方法通常用于把一个数组转换成一个（发送多个值的）可观察对象。
下面是一个简单的创建可观察对象，并订阅它的例子：

```TypeScript
// 使用of()方法创建一个可观察对象实例。
const myObservable = of(1, 2, 3);

// 创建观察者对象，并定义三种处理器，用来处理接收到的信息。
const myObserver = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};

// 调用的可观察对象的subscribe()方法，传入带有处理消息方法的观察者 myObserver。
myObservable.subscribe(myObserver);
// Logs:
// Observer got a next value: 1
// Observer got a next value: 2
// Observer got a next value: 3
// Observer got a complete notification
```
subscribe()方法还可以接收定义在同一行中的回调函数。下面的subscribe()调用和上面预定义观察者的列子是等价的。
```TypeScript
myObservable.subscribe(
  x => console.log('Observer got a next value: ' + x),
  err => console.error('Observer got an error: ' + err),
  () => console.log('Observer got a complete notification')
);
```
* next() 函数可以接受消息字符串、事件对象、数字值或各种结构，具体类型取决于上下文。 为了更通用一点，我们把由可观察对象发布出来的数据统称为流。任何类型的值都可以表示为可观察对象，而这些值会被发布为一个流。

#### 创建可观察者对象

使用Observable构造函数可以创建任何类型的可观察流。这构造函数接收一个订阅函数，这个订阅函数又接收一个Observer对象。当执行可观察对象的subscribe()方法时，就会执行这个订阅函数。

```TypeScript
// This function runs when subscribe() is called
function sequenceSubscriber(observer) {
  // synchronously deliver 1, 2, and 3, then complete
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();

  // unsubscribe function doesn't need to do anything in this
  // because values are delivered synchronously
  return {unsubscribe() {}};
}

// Create a new Observable that will deliver the above sequence
const sequence = new Observable(sequenceSubscriber);

// execute the Observable and print the result of each notification
sequence.subscribe({
  next(num) { console.log(num); },
  complete() { console.log('Finished sequence'); }
});

// Logs:
// 1
// 2
// 3
// Finished sequence
```
下面例子中创建了一个用来发布事件的可观察对象，订阅函数通过内联的方式定义，并使用这个函数创建发布keydown事件的可观察对象：
```TypeScript
function fromEvent(target, eventName) {
  return new Observable((observer) => {
    const handler = (e) => observer.next(e);

    // Add the event handler to the target
    target.addEventListener(eventName, handler);

    return () => {
      // Detach the event handler from the target
      target.removeEventListener(eventName, handler);
    };
  });
}


const ESC_KEY = 27;
const nameInput = document.getElementById('name') as HTMLInputElement;

const subscription = fromEvent(nameInput, 'keydown')
  .subscribe((e: KeyboardEvent) => {
    if (e.keyCode === ESC_KEY) {
      nameInput.value = '';
    }
  });
```

#### 多播
* 多播用来让可观察对象在一次执行中同时广播给多个订阅者。借助支持多播的可观察对象，你不必注册多个监听器，而是复用第一个（next）监听器，并且把值发送给各个订阅者。当创建可观察对象时，决定你希望别人怎么用这个对象以及是否对它的值进行多播。

#### 错误处理
* 由于可观察对象会异步生成值，所以用 try/catch 是无法捕获错误的。你应该在观察者中指定一个 error 回调来处理错误。发生错误时还会导致可观察对象清理现有的订阅，并且停止生成值。可观察对象可以生成值（调用 next 回调），也可以调用 complete 或 error 回调来主动结束。

## RxJs库

>响应式编程是一种面向数据流和变更传播的 **异步编程方式**。RxJs(Reactive Extensions for JavaScript)是一个使用 **可观察对象**进行响应式编程的库，它让组合异步代码和基于回调的代码变得更简单。

RxJS提供了对Observable类型的实现，同时还提供了一些工具函数用于：
1.把现有的异步代码转换成可观察对象
2.迭代流中的各个值
3.把这些值映射成其他类型
4.对流进行过滤
5.组合多个流

#### 创建可观察对象的函数

RxJS提供了一些用来创建可观察对象的函数：
* 从promise创建可观察对象：fromPromise()
* 从counter创建可观察对象：const secondsCounter = interval(1000);
* 从event创建可观察对象：fromEvent()
* 从AJAX请求创建可观察对象：ajax()

#### 操作符(Operators)

操作符是基于可观察对象构建的一些对集合进行复杂操作的函数。操作符接受一些配置项，然后返回一个以可观察对象为参数的函数。当执行这个返回的函数时，这个操作符会观察可观察对象中发出的值，转换它们，并返回**由转换后的值组成的新的可观察对象**。下面是一个操作组map()的例子：
```TypeScriptGGggG
import { map } from 'rxjs/operators';

const nums = of(1, 2, 3);

const squareValues = map((val: number) => val * val);
const squaredNums = squareValues(nums);

squaredNums.subscribe(x => console.log(x));

// Logs
// 1
// 4
// 9
```
上面例子中，操作符map将可观察对象nums流出的值，转换为改值的平方。

#### 管道
管道可以把多个操作符返回的函数组成一个。pipe()函数你要组合的操作符为函数，并返回一个新的函数，当执行这个新函数时，就会顺序执行那些被组合进去的操作符函数。下面这个例子中pipe()函数将filter和map两个操作符函数组合起来：
```TypeScript
import { filter, map } from 'rxjs/operators';
const nums = of(1, 2, 3, 4, 5);
// pipe接收的参数为操作符，pipe函数返回的函数 仍然将可观察对象作为参数.
const squareOddVals = pipe(
  filter((n: number) => n % 2 !== 0),
  map(n => n * n)
);
// squareOddVals函数仍然返回一个可观察对象，并且该可观察对象在subscribe时，流出的值会依次进行filte和map
const squareOdd = squareOddVals(nums);

squareOdd.subscribe(x => console.log(x));
```
pipe()函数同时也是RxJS中Observable上的一个方法，所以可以如下简写：
```TypeScript
import { filter, map } from 'rxjs/operators';

const squareOdd = of(1, 2, 3, 4, 5)
  .pipe(
    filter(n => n % 2 !== 0),
    map(n => n * n)
  );

// Subscribe to get values
squareOdd.subscribe(x => console.log(x));
```
#### 常用操作符
1. 创建：from,fromPromise,fromEvent,of
2. 组合：combineLatest, concat, merge, startWith , withLatestFrom, zip
3. 过滤：debounceTime, distinctUntilChanged, filter, take, takeUntil
4. 转换：bufferTime, concatMap, map, mergeMap, scan, switchMap
5. 工具：tap
6. 多播：share

#### 错误处理

RxJS提供了catchError操作符，它允许你在管道中处理已知错误。如果捕获这个错误并提供一个默认值，流就会处理这些值，而不会报错。如下所示：
```TypeScript
import { ajax } from 'rxjs/ajax';
import { map, catchError } from 'rxjs/operators';
// Return "response" from the API. If an error happens,
// return an empty array.
const apiData = ajax('/api/data').pipe(
  map(res => {
    if (!res.response) {
      throw new Error('Value expected!');
    }
    return res.response;
  }),
  catchError(err => of([]))
);

apiData.subscribe({
  next(x) { console.log('data: ', x); },
  error(err) { console.log('errors already caught... will not run'); }
});
```
#### 重试失败的可观察对象
retry操作符可以重新尝试失败的请求。可以在catchError之前使用retry操作符。它会订阅原始的可观察对象，重新运行导致结果出错的动作序列。如果其中包含HTTP请求，就会重新发起那个HTTP请求。

#### 可观察对象的命名约定

* Angular中可观察对象的名字以 **“$”** 符号结尾。如果需要用某个属性来存储来自可观察对象的最近一个值，它的命名惯例是与可观察对象同名，但不带“$”后缀。

## Angular中的可观察对象
Angular使用可观察对象作为处理各种常用一步操作的接口。比如：
* EventEmitter类派生自Obervable。
* HTTP模板使用可观察对象处理AJAX请求和响应。
* 路由器和表单模块使用可观察对象来监听对用户输入事件的响应。
