---
layout: post
title: "JavaScript 面向对象"
date: 2018-12-15
author: "WangX"
catalog: true
header-style: text
tags:
    - JavaScript
---

## 对象
>JavaScript中的对象是可变的键控集合(key collections)。    

* 对象是一个容器，封装了property和method。property是对象的状态，method是对象的行为。
* JavaScript 语言的对象体系，不是基于“类”的，而是基于构造函数（constructor）和原型链（prototype）。
#### 构造函数
* 构造函数描述实例对象的基本结构，是对象的模板。构造函数的名字第一个字母通常大写。
* 构造函数的两个特点：
1. 函数体内部使用了this关键字，**代表所有生成的对象实例**。
2. 生成对象的时，必须使用new命令。
#### new
* new命令的作用就是执行构造函数。    

```javascript
var Car = function（p）{
    this.val = p;
}

var car = new Car(100);

```
* 使用new命令时，后面的函数依次执行：
1. 创建一个空对象，作为将要返回的新对象实例。
2. 将这个空对象的原型，指向构造函数的prototype属性。
3. 将这个空对象赋值给函数内部的this关键字。
4. 执行构造函数内部的代码。
* 构造函数内部的this**指向新生成的对象**，所有针对this的操作，都会发生在这个空对象上。
* 构造函数就是操作一个空对象（this对象），将其“构造”为需要的样子。
* 如果构造函数内部有return语句，而且return后面跟着一个对象，new命令会返回return语句指定的对象；否则，就会不管return语句，返回this对象。   

```javascript
var Vehicle = function (){
  this.price = 1000;
  return { price: 2000 };
};

(new Vehicle()).price // 200
```
* 如果对普通函数（内部没有this关键字的函数）使用new命令，则会返回一个空对象。
* 函数内部可以使用new.target属性。如果当前函数是new命令调用，new.target指向当前函数，否则为undefined。使用这个属性可以判断函数调用的时候，是否是使用new命令。 

```javascript
function f() {
  if (!new.target) {
    throw new Error('请使用 new 命令调用！');
  }
}

f() // Uncaught Error: 请使用 new 命令调用！
```

#### Object.create()创建实例对象
* 没有构造函数，使用现有的对象来提供新创建的对象可以使用Obejct.create()函数。

## 关键字this
* 关键字this总是返回一个对象，代表属性或者方法“当前”所在的对象。this的值取决于调用的模式。
* 函数可以作为对象的属性，也可以独立存在。即函数可以在不同的上下文环境中执行。**所以需要有一种机制，能够在函数体内部获得当前的context。this设计的目的就是在函数体内部，指代函数当前的运行环境**。          

```javascript
var f = function () {
  console.log(this.x);
}

var x = 1;
var obj = {
  f: f,
  x: 2,
};

f() // 1  单独执行

obj.f() // 2  obj 环境执行
```
上述代码中，函数f在全局环境执行，this.x指向全局环境的x；在obj环境执行，this.x指向obj.x。
## 不通场景中的this
#### 函数调用
* 函数调用模式(全局环境)，当一个函数并非一个对象的属性时，它被当作一个函数来调用。此时this指向全局对象。 此外，全局环境中使用this，它指向的就是顶层对象window。      

```javascript
function f() {
  console.log(this === window);
}
f() // true
this === window //true
```
#### 构造函数调用模式
* 构造函数中的this，指的是新的实例对象。
#### 方法调用模式
>当一个函数被保存为对象的一个属性时，称之为一个方法。
* 如果对象的方法里面包含this，this的指向就是方法运行时所在的对象。该方法赋值给另一个对象，就会改变this的指向。下面代码中，obj.foo方法执行时，this指向obj。    

```javascript
var obj ={
  foo: function () {
    console.log(this);
  }
};

obj.foo() // obj
```        
* 下面代码中obj.foo就是一个值。调用时运行环境已经不是obj了，而是全局环境，所以this不再指向obj，而是window。     

```javascript
(obj.foo = obj.foo)() // window 和下面等价

(obj.foo = function () {
  console.log(this);
})()

```   
* 以这样理解，JavaScript 引擎内部，obj和obj.foo储存在两个内存地址，称为地址一和地址二。obj.foo()这样调用时，是从地址一调用地址二，因此地址二的运行环境是地址一，this指向obj。但是，上面是直接取出地址二进行调用，这样的话，运行环境就是全局环境，因此this指向全局环境。

* 使用一个变量固定this的值，然后内层函数调用这个变量，是非常常见的做法。

## 绑定this的方法
>JavaScript 提供了call、apply、bind这三个方法，来切换/固定this的指向。

#### Function.prototype.call(thisArg, arg1, arg2, ...)
>call() 方法**调用**一个函数, 其具有一个指定的this值和分别地提供的参数(参数的列表)。

* 函数实例的call方法，可以指定函数内部this的指向（即函数执行时所在的作用域），然后在所指定的作用域中，调用该函数。    

```javascript
var obj = {};

var f = function () {
  return this;
};

f() === window // true
f.call(obj) === obj // true
```    
* 上面代码中，全局环境运行函数f时，this指向全局环境（浏览器为window对象）；call方法可以改变this的指向，指定this指向对象obj，然后在对象obj的作用域中运行函数f。
* call方法的参数，应该是一个对象。如果参数为空、null和undefined，则默认传入全局对象。如果call方法的参数是一个原始值，那么这个原始值会自动转成对应的包装对象，然后传入call方法。
* call的第一个参数就是this所要指向的那个对象，后面的参数则是函数调用时所需的参数。      

```javascript
function add(a, b) {
  return a + b;
}
add.call(this, 1, 2) // 
```   
#### Function.prototype.apply(thisValue, [arg1, arg2, ...])

* apply方法的作用与call方法类似，也是改变this指向，然后再调用该函数。**唯一**的区别就是，它接收一个数组作为函数执行时的参数。第二个参数则是一个数组，该数组的所有成员依次作为参数，传入原函数。原函数的参数，在call方法中必须一个个添加，但是在apply方法中，必须以数组形式添加。

#### Function.prototype.bind(thisArg[, arg1[, arg2[, ...]]]) 

* bind方法用于将函数体内的this绑定到某个对象，然后返回一个新函数。

## 对象的继承
> 大部分面向对象的语言，都是通过“类”(class)实现对象的继承。传统上，JavaScript的继承不通过class，而是通过“原型对象”(prototype)实现。ES6中引入了class语法，可以通过class实现继承。

#### 构造函数的缺点
* 构造函数可以当做对象的模板，实例对象的属性和方法可以定义在构造函数内部。**但是，同一构造函数的多个实例之间，无法共享属性**。       

```javascript
function Cat(name, color) {
  this.name = name;
  this.color = color;
  this.meow = function () {
    console.log('喵喵');
  };
}

var cat1 = new Cat('大毛', '白色');
var cat2 = new Cat('二毛', '黑色');

cat1.meow === cat2.meow  //false
```     
上面代码中，cat1和cat2是Cat构造函数的两个实例，都具有meow方法。由于meow方法是生成在每个实例对象上面，所以两个实例就生成了两次。也就是说，每新建一个实例，就会新建一个meow方法。这既没有必要，又浪费系统资源，因为所有meow方法都是同样的行为，完全应该共享。
* JavaScript中使用原型对象prototype来解决这个问题。

#### prototype 属性的作用
* JavaScript 继承机制的设计思想就是，**原型对象的所有属性和方法，都能被实例对象共享。**
* JavaScript 规定，每个函数都有一个prototype属性，指向一个对象。对于普通函数来说，该属性基本无用。但是，**对于构造函数来说，生成实例的时候，该属性会自动成为实例对象的原型。**     

```javascript
function Animal(name) {
  this.name = name;
}
Animal.prototype.color = 'white';

var cat1 = new Animal('大毛');
var cat2 = new Animal('二毛');

cat1.color // 'white'
cat2.color // 'white'
```     
上面代码中，构造函数Animal的prototype属性，就是实例对象cat1和cat2的原型对象。原型对象上添加一个color属性，实例对象都共享了该属性。
* 原型对象的属性不是实例对象自身的属性。只要修改原型对象，变动就立刻会体现在所有实例对象上。
* 当实例对象本身没有某个属性或者方法时，就会到原型对象对寻找该属性或方法。如果实例对象自身存在这个属性或者方法，就不会再去原型对象寻找。
* 原型对象的作用，就是定义所有实例对象共享的属性和方法。这也是它被称为原型对象的原因，而实例对象可以视作从原型对象衍生出来的子对象。

#### 原型链

* JavaScript 规定，所有对象都有自己的原型对象（prototype）。一方面，任何一个对象，都可以充当其他对象的原型；另一方面，由于原型对象也是对象，所以它也有自己的原型。因此，就会形成一个“原型链”（prototype chain）：对象到原型，再到原型的原型……
* 所有对象的原型都可以上溯到Object.prototype,即构造函数prototype属性。Object.prototype的原型是null。即原型链的尽头是null。
* 读取对象的某个属性时，JavaScript 引擎先寻找对象本身的属性，如果找不到，就到它的原型去找，如果还是找不到，就到原型的原型去找。如果直到最顶层的Object.prototype还是找不到，则返回undefined。如果对象自身和它的原型，都定义了一个同名属性，那么优先读取对象自身的属性，这叫做“覆盖”（overriding）。

#### constructor 属性
* prototype对象有一个constructor属性，默认指向prototype对象所在的构造函数。由于constructor属性定义在prototype对象上面，意味着可以被所有实例对象继承。
* constructor属性的作用：
1. 可以得知某个实例对象，是由哪个构造函数产生的。
2. 有了constructor属性，就可以从一个实例对象新建另一个实例。    

```javascript
function Constr() {}
var x = new Constr();

var y = new x.constructor();
y instanceof Constr // true
```   
上面代码中，x是构造函数Constr的实例，可以从x.constructor间接调用构造函数。
* constructor属性表示原型对象与构造函数之间的关联关系，**如果修改了原型对象，一般会同时修改constructor属性**，防止引用的时候出错。
#### 构造函数的继承
* instanceof运算符返回一个布尔值，表示对象是否为某个构造函数的实例。
* 一个构造函数继承另外一个构造函数，可以分为两步实现：
1. 在子类的构造函数中，调用父类的构造函数。下面代码中，Sub是子类的构造函数，this是子类的实例。在实例上调用父类的构造函数Super，就会让子类实例具有父类实例的属性。   

```javascript
function Sub(value) {
    Super.call(this);
    this.prop = value;
}
```
2. 将子类的原型指向父类的原型，这样子类就可以继承父类原型。下面代码中，Sub.prototype是子类的原型，将它赋值为Object.create(Super.prototype),而不是直接等于Super.prototype。否则对Sub.prototype的操作，会连父类的原型一起修改掉。      

```javascript
Sub.prototype = Object.create(Super.prototype);
Sub.prototype.constructor = Sub;
Sub.prototype.method = '...';
```
* 有时只需要单个方法的继承，这时可以采用下面的写法:       

```javascript
ClassB.prototype.print = function() {
    ClassA.prototype.print.call(this);
}
```

#### 多重继承



