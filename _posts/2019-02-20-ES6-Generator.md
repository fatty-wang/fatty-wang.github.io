---
layout: post
title: "ES6中的Generator"
date: 2019-02-20
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
    - ES6
---

## 概念
* Generator 函数是一个状态机，封装了多个内部状态。
* Generator 函数是一个Iterator对象生成函数，返回一个Iterator，可以一次遍历Generator函数内部的每一个状态。
* Generator函数有两个特征：
1. function关键字与函数名**之间**有一个星号(*)
2. 函数体内部使用yield表达式，定义不同的内部状态

```javascript
//定义Generator函数helloWorld，内有两个yield表达式
//有三个状态：hello，world和return语句。
function* helloWorld() {
    yield 'hello';
    yield 'world';
    return 'ending';
}

var hw = hellWorld();

hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```
* 调用 Generator 函数后，该函数**并不执行**，**返回**的也不是函数运行结果，而是一个指向内部状态的指针对象（**Iterator** Object）。
* 每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。
* Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。
* 上面代码中，第一次调用next方法，Generator 函数开始执行，直到遇到第一个yield表达式为止。**next方法返回一个对象，它的value属性就是当前yield表达式的值hello，done属性的值false，表示遍历还没有结束。**
* 第二次调用，Generator 函数从上次yield表达式停下的地方，一直执行到下一个yield表达式。
* 第三次调用，Generator 函数从上次yield表达式停下的地方，一直执行到return语句（如果没有return语句，就执行到函数结束）。**next方法返回的对象的value属性，就是紧跟在return语句后面的表达式的值（如果没有return语句，则value属性的值为undefined），done属性的值true，表示遍历已经结束。**
* 第四次调用，此时 Generator 函数已经运行完毕，next方法返回对象的value属性为undefined，done属性为true。以后再调用next方法，返回的都是这个值

* 每次调用Iterator对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。

## yield
* yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。下面yield后面的表达式1+1，不会立即求值，只会在next方法将指针移到这句时，才会求值。

```javascript
function*gen() {
    yield 1+1;
}
```
* Generator 函数可以不用yield表达式，这时就变成了一个单纯的暂缓执行函数。下面，函数f如果是普通函数，在为变量generator赋值时就会执行。但是，函数f是一个 Generator 函数，就变成只有调用next方法时，函数f才会执行。

```javascript
function* f() {
  console.log('执行了！')
}

var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);
```

* yield表达式**只能**用在 Generator 函数里面，用在其他地方都会报错。
* yield表达式如果用在另一个表达式之中，必须放在圆括号里面。yield表达式用作**函数参数**或放在**赋值表达式的右边**，可以不加括号。

## 与Iterator
* Generator 函数就是Iterator生成函数，因此可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口。
* Generator 函数执行后，返回一个Iterator对象。该对象本身也具有Symbol.iterator属性，执行后返回自身。

```javascript
function* gen(){
  // some code
}

var g = gen();

g[Symbol.iterator]() === g
// true
```
## next方法的参数
* **yield表达式本身没有返回值，或者说总是返回undefined**。
* next方法可以带一个参数，该参数被当做**上一个yield**表达式的返回值。
* Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过next方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。
* 由于next方法的参数表示上一个yield表达式的返回值，所以**在第一次使用next方法时，传递参数是无效的**。V8 引擎直接忽略第一次使用next方法时的参数，只有从第二次使用next方法开始，参数才是有效的。从语义上讲，第一个next方法用来启动Iterator对象，所以不用带有参数。
*
## for...of
* for...of循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用next方法。

```javascript
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```
* 一旦next方法的返回对象的done属性为true，for...of循环就会中止，且不包含该返回对象，所以上面代码的**return语句**返回的6，**不包括在for...of循环之中**。

* Generator函数和for...of循环，实现斐波那契数列的例子。

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```
## Generator.prototype.throw()
* Generator 函数返回的Iterator对象，都有一个throw方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获。

```javascript
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('内部捕获', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b
```
上面代码中，Iterator对象i连续抛出两个错误。第一个错误被 Generator 函数体内的catch语句捕获。i第二次抛出错误，由于 Generator 函数内部的catch语句已经执行过了，不会再捕捉到这个错误了，所以这个错误就被抛出了 Generator 函数体，被函数体外的catch语句捕获。

* 如果 Generator 函数内部没有部署try...catch代码块，那么throw方法抛出的错误，将被外部try...catch代码块捕获。
* 如果 Generator 函数内部和外部，都没有部署try...catch代码块，那么程序将报错，直接中断执行。
* throw方法抛出的错误要被内部捕获，前提是必须**至少执行过一次next方法**。
* throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。下面代码中，g.throw方法被捕获以后，自动执行了一次next方法，所以会打印b。另外，也可以看到，只要 Generator 函数内部部署了try...catch代码块，那么Iterator的throw方法抛出的错误，不影响下一次遍历。

```javascript
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```
* 一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。如果此后还调用next方法，将返回一个value属性等于undefined、done属性等于true的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了。

## Generator.prototype.return()
* Generator 函数返回的Iterator对象，还有一个return方法，可以返回给定的值，并且终结遍历 Generator 函数。下面代码中，Iterator对象g调用return方法后，返回值的value属性就是return方法的参数foo。并且，Generator 函数的遍历就终止了，返回值的done属性为true，以后再调用next方法，done属性总是返回true。

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```
* 如果return方法调用时，不提供参数，则返回值的value属性为undefined。
* 如果 Generator 函数内部有try...finally代码块，且正在执行try代码块，那么return方法会推迟到finally代码块执行完再执行。下面代码中，调用return方法后，就开始执行finally代码块，然后等到finally代码块执行完，再执行return方法。

```javascript
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers();
g.next() // { value: 1, done: false }
g.next() // { value: 2, done: false }
g.return(7) // { value: 4, done: false }
g.next() // { value: 5, done: false }
g.next() // { value: 7, done: true }
```
* ext()、throw()、return()作用都是让 Generator 函数恢复执行，并且使用不同的语句替换yield表达式。next()是将yield表达式替换成一个值。throw()是将yield表达式替换成一个throw语句。return()是将yield表达式替换成一个return语句。

## yield* 表达式
* 如果yield表达式后面跟的是一个Iterator对象，需要在yield表达式后面加上星号，表明它返回的是一个Iterator对象。这被称为yield*表达式。
* yield*表达式用来在一个 Generator 函数里面执行另一个 Generator 函数。如下：

```javascript
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y
```
* yield*后面的 Generator 函数（没有return语句时），等同于在 Generator 函数内部，部署一个for...of循环。
* 何数据结构只要有 Iterator 接口，就可以被yield*遍历。)
* 使用yield*语句遍历完全二叉树：

```javascript
// 下面是二叉树的构造函数，
// 三个参数分别是左树、当前节点和右树
function Tree(left, label, right) {
  this.left = left;
  this.label = label;
  this.right = right;
}

// 下面是中序（inorder）遍历函数。
// 由于返回的是一个遍历器，所以要用generator函数。
// 函数体内采用递归算法，所以左树和右树要用yield*遍历
function* inorder(t) {
  if (t) {
    yield* inorder(t.left);
    yield t.label;
    yield* inorder(t.right);
  }
}

// 下面生成二叉树
function make(array) {
  // 判断是否为叶节点
  if (array.length == 1) return new Tree(null, array[0], null);
  return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

// 遍历二叉树
var result = [];
for (let node of inorder(tree)) {
  result.push(node);
}

result
// ['a', 'b', 'c', 'd', 'e', 'f', 'g']
```
## 作为对象属性的 Generator 函数

```javascript
let obj = {
  * myGeneratorMethod() {
    ···
  }
}
//等价于
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```
## Generator函数的this
* Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也**继承了 Generator 函数的prototype对象上的方法**。

```javascript
function* g(){
    this.a = 11;
}

g.prototype.hello = function() {
    return 'hi!';
};

let obj = g();

obj instanceof g //true
obj.hello() // 'hi!'
obj.a // undefined
```
* Generator 函数g返回的遍历器obj，是g的实例，而且继承了g.prototype。但是，如果把g当作普通的构造函数，并不会生效，因为g返回的总是遍历器对象，而不是this对象, obj对象拿不到这个属性。
* Generator 函数也不能跟new命令一起用，会报错。
* 将Gneerator内部的this对象绑定到 prototype对象上，就可以访问this.

```javascript
function* F() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}
var f = F.call(F.prototype);

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```
## Generator 与状态机
* Generator 之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停态。
## Generator 与协程 
* 协程（coroutine）是一种程序运行的方式，可以理解成“协作的线程”或“协作的函数”。
* 协程既可以用单线程实现，也可以用多线程实现。前者是一种特殊的子例程，后者是一种特殊的线程。
#### 协程与子例程的差异
* 传统的“子例程”（subroutine）采用**堆栈式“后进先出”**的执行方式，只有当调用的子函数完全执行完毕，才会结束执行父函数。
* 协程与其不同，多个线程（单线程情况下，即多个函数）可以并行执行，但是只有一个线程（或函数）处于正在运行的状态，其他线程（或函数）都处于暂停态（suspended），线程（或函数）之间可以交换执行权。这种可以并行执行、交换执行权的线程（或函数），就称为协程。
* 在内存中，子例程只使用一个栈（stack），而协程是同时存在多个栈，但只有一个栈是在运行状态，也就是说，**协程是以多占用内存为代价**，实现多任务的并行。
#### 协程与普通线程的差异
* 协程适合用于多任务运行的环境。在这个意义上，它与普通的线程很相似，都有自己的执行上下文、可以分享全局变量。
* 不同之处，同一时间可以有多个线程处于运行状态，但运行的协程只能有一个，其他协程都处于暂停状态。
* 普通的线程是抢先式的，到底哪个线程优先得到资源，必须由运行环境决定，但是协程是合作式的，执行权由协程自己分配。
* JavaScript 是单线程语言，只能保持一个调用栈。引入协程以后，每个任务可以保持自己的调用栈。这样做的最大好处，就是抛出错误的时候，可以找到原始的调用栈。不至于像异步操作的回调函数那样，一旦出错，原始的调用栈早就结束。
* Generator 函数是 ES6 对协程的实现，但属于不完全实现。Generator 函数被称为“半协程”（semi-coroutine），意思是只有 Generator 函数的调用者，才能将程序的执行权还给 Generator 函数。如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。
* 如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用yield表达式交换控制权。

## Gennerator与上下文
* JavaScript 代码运行时，会产生一个全局的上下文环境（context，又称运行环境），包含了当前所有的变量和对象。然后，执行函数（或块级代码）的时候，又会在当前上下文环境的上层，产生一个函数运行的上下文，变成当前（active）的上下文，由此形成一个上下文环境的堆栈（context stack）。
* 这个堆栈是“后进先出”的数据结构，最后产生的上下文环境首先执行完成，退出堆栈，然后再执行完成它下层的上下文，直至所有代码执行完成，堆栈清空。
* Generator 函数不是这样，它执行产生的上下文环境，一旦遇到yield命令，就会**暂时退出堆栈，但是并不消失**，里面的所有变量和对象会冻结在当前状态。等到对它执行next命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行。

## 应用
* Generator可以暂停函数执行，返回任意表达式的值。
#### 异步操作的同步化表达
* Generator 函数的暂停执行的效果，意味着可以把异步操作写在yield表达式里面，等到调用next方法时再往后执行。而异步操作的**后续步骤可以放在yield表达式下面**，等待yield表达式中的异步操作执行结束后，再执行。如下：第一次调用loadUI函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用next方法，则会显示Loading界面（showLoadingScreen），并且异步加载数据（loadUIDataAsynchronously）。等到数据加载完成，再一次使用next方法，则会隐藏Loading界面。

```javascript
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
var loader = loadUI();
// 加载UI
loader.next()

// 卸载UI
loader.next()
```
* Ajax 是典型的异步操作，通过 Generator 函数部署 Ajax 操作，可以用同步的方式表达。

```javascript
function* main() {
  var result = yield request("http://some.url");
  var resp = JSON.parse(result);
    console.log(resp.value);
}

function request(url) {
  makeAjaxCall(url, function(response){
    it.next(response);
  });
}

var it = main();
it.next();
```
上面代码中，next方法的调用触发request方法，而在request方法中的makeAjaxCall中再次调用next方法，并传入response，因为yield表达式本身没有值，必须通过next赋值。

#### 控制流管理
#### 部署 Iterator 接口
#### 作为数据结构

## Generator 函数的数据交换和错误处理
* Generator 函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。除此之外，它还有两个特性，使它可以作为异步编程的完整解决方案：**函数体内外的数据交换**和**错误处理机制**。
* Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。这意味着，出错的代码与处理错误的代码，实现了时间和空间上的分离，这对于异步编程无疑是很重要的。