---
layout: post
title: "JavaScript拾遗"
date: 2018-11-01 
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
* delete命令用于删除对象的属性，删除成功后返回true。   
```javascript
var obj = { p: 1 };
Object.keys(obj) // ["p"]

delete obj.p // true
obj.p // undefined
Object.keys(obj) // []
```     

注意，删除一个不存在的属性，delete不报错，而且返回true。    

```javascript
var obj = {};
delete obj.p //true
```  

只有一种情况，delete命令会返回false，那就是该属性存在，且不得删除。
* delete只能删除对象本身的属性，无法删除继承的属性。
* in 运算符用于检查对象是否包含某个key。它的左边是一个字符串，表示属性名，右边是一个对象。

```javascript
var obj = { p: 1 };
'p' in obj // true
'toString' in obj // true
```    

in运算符不能识别哪些属性是对象自身的，哪些属性是继承的。可以使用对象的hasOwnProperty方法判断。

* for...in循环用来遍历一个对象的全部属性。遍历的是对象所有可遍历（enumerable）的属性，会跳过不可遍历的属性。不仅遍历对象自身的属性，还遍历继承的属性。

#### 函数

* 函数的三种声明方法：
1. function命令声明函数，function命令后面是函数名，函数名后面是一对圆括号，里面是传入函数的参数。函数体放在大括号里面。下面的代码命名了一个print函数，以后使用print()这种形式，就可以调用相应的代码。    
```javascript
function print(s) {
  console.log(s);
}
```    
2. 函数表达式。这种写法将一个匿名函数赋值给变量。采用函数表达式声明函数时，function命令后面不带有函数名。如果加上函数名，该函数名只在函数体内部有效，在函数体外部无效。    
```javascript
var print = function(s) {
  console.log(s);
}
``` 
下面的声明非常常见，这种写法的用处有两个，一是可以在函数体内部调用自身，二是方便除错（除错工具显示函数调用栈时，将显示函数名，而不再显示这里是一个匿名函数）。**需要注意的是，函数的表达式需要在语句的结尾加上分号，表示语句结束。而函数的声明在结尾的大括号后面不用加分号。**    
```javascript
var f = function f() {};
}
```
3. Function构造函数。 Function构造函数接受三个参数，除了最后一个参数是函数的“函数体”，其他参数都是函数的参数。只有最后一个参数会被当做函数体，如果只有一个参数，该参数就是函数体。   
```javascript
var add = new Function(
  'x',
  'y',
  'return x + y'
); // 等同于
function add(x, y) {
  return x + y;
}
}
```

* 如果同一个函数被多次声明，后面的声明就会覆盖前面的声明。
* JavaScript将函数为第一等公民，与其他数据类型相等。
* JavaScript 引擎将函数名视同变量名，所以采用function命令声明函数时，整个函数会像变量声明一样，被提升到代码头部。所以，下面的代码不会报错。
* 但是如果采用赋值语句定义函数，如下：  
```javascript
f();
var f = function (){};
// TypeError: undefined is not a function
}
```   
上面的代码等同于：
```javascript
var f;
f();
f = function () {};
```
上面代码第二行，调用f的时候，f只是被声明了，还没有被赋值，等于undefined，所以会报错。

* 如果同时采用function命令和赋值语句声明同一个函数，最后总是采用赋值语句的定义。    

* 函数name属性返回函数的名字。函数的length属性返回函数**预期**传入的参数个数。toString()方法返回函数的源码。

* 在ES5中，JavaScript只有两种作用域：全局作用域和函数作用域。**ES6中又增加了块级作用域**。
* 函数的参数的个数没有限制，可以多传或者少传。如果想要省略靠前的参数，只有显示的传入undefined。
* 函数参数如果是原始类型的值（数值、字符串、布尔值），传递方式是传值传递（passes by value）。在函数体内修改参数值，不会影响到函数外部。
* 如果函数参数是复合类型的值（数组、对象、其他函数），传递方式是传址传递（pass by reference）。传入函数的原始值的地址，因此在函数内部修改参数，将会影响到原始值。
* 如果有同名的参数，则取最后出现的那个值。
* arguments对象包含了函数运行时的所有参数，arguments[0]就是第一个参数，arguments[1]就是第二个参数，以此类推。这个对象只有在函数体内部，才可以使用。由于 JavaScript 允许函数有不定数目的参数，所以需要一种机制，可以在函数体内部读取所有参数。
* 虽然arguments很像数组，但它是一个对象。数组专有的方法（比如slice和forEach），不能在arguments对象上直接使用。

* **闭包需要单独一篇**

#### 数组

* 数组属于一种特殊的对象。typeof运算符会返回数组的类型是object。
* 数组的特殊性体现在，它的键名是按次序排列的一组整数（0，1，2...）。
* 对于数值的键名，不能使用点结构。
```javascript
var arr = [1,2,3];
arr.0 //SyntaxError
```     

* 数组的length属性，返回数组的成员数量。length属性是可写的。如果人为设置一个小于当前成员个数的值，该数组的成员会自动减少到length设置的值。
```javascript
var arr = [ 'a', 'b', 'c' ];
arr.length // 3
arr.length = 2;
arr // ["a", "b"]
```
* 清空数组的一个有效方法，就是将length属性设为0。如果人为设置length大于当前元素个数，则数组的成员数量会增加到这个值，新增的位置都是空位。当length属性设为大于数组个数时，读取新增的位置都会返回undefined。
* 当数组的某个位置是空元素，即两个逗号之间没有任何值，我们称该数组存在空位（hole）。数组的空位不影响length属性。数组的空位是可以读取的，返回undefined。
* 使用delete命令删除一个数组成员，会形成空位，并且不会影响length属性。
* length属性不过滤空位。所以，使用length属性进行数组遍历，一定要非常小心。
* 数组的某个位置是空位，与某个位置是undefined，是不一样的。如果是空位，使用数组的forEach方法、for...in结构、以及Object.keys方法进行遍历，空位都会被跳过。
* 类似数组的对象: 一个对象的所有键名都是正整数或零，并且有length属性，那么这个对象就很像数组.


## 运算符

#### 算数运算符
* 指数运算符（**）完成指数运算，前一个运算子是底数，后一个运算子是指数。
#### 比较运算符

* 字符串按照字典顺序进行比较。JavaScript 引擎内部首先比较首字符的 Unicode 码点。如果相等，再比较第二个字符的 Unicode 码点，以此类推。所有字符都有Unicode码点，汉字也可以比较。

* 非字符串比较：
1. 原始类型的值，先转换成数值在比较。
2. 对象会转为原始类型的值，再进行比较。对象转换成原始类型的值，算法是先调用valueOf方法；如果返回的还是对象，再接着调用toString方法。
* 相等运算符（==）比较两个值是否相等，严格相等运算符（===）比较它们是否为“同一个值”。如果两个值不是同一类型，严格相等运算符（===）直接返回false，而相等运算符（==）会将它们转换成同一个类型，再用严格相等运算符进行比较。
* 两个复合类型（对象、数组、函数）的数据比较时，不是比较它们的值是否相等，而是比较它们是否指向同一个地址。如果两个变量引用同一个对象，则它们相等。
```javascript
var v1 = {};
var v2 = v1;
v1 === v2; //true
```
* 对于两个对象的比较，严格相等运算符比较的是地址，而大于或小于运算符比较的是值。
* undefined和null与自身严格相等。
* 由于变量声明后默认值是undefined，因此两个只声明未赋值的变量是相等的。
* 相等运算符：undefined和null与其他类型的值比较时，结果都为false，它们互相比较时结果为true。
* 最好只使用严格相等运算符（===）。

#### 其他运算符
* void运算符的作用是执行一个表达式，然后不返回任何值，或者说返回undefined。
* 逗号运算符用于对两个表达式求值，并返回后一个表达式的值。
```javascript
'a', 'b' // "b"
var x = 0;
var y = (x++, 10);
x // 1
y // 10
```

## 数据类型转换
* JavaScript 是一种动态类型语言，变量没有类型限制，可以随时赋予任意值。

#### 强制转换

* 使用Number()、String()和Boolean()三个函数，手动将各种类型的值分别转换成数字、字符串或布尔值。

#### 自动转换
* 下面三种情况JavaScript 会自动转换数据类型：
1. 不同类型的数据相互运算
2. 对非布尔值类型的数据求布尔值
3. 对非数值类型的值使用一元运算符（+和-）
* 除去undefined、null、+0、-0、NaN和''会自动转为false外，其他全部是true。
* 自动转换是JavaScript自动调用Number()、String()和Boolean()三个函数。

## 错误处理机制
#### Error实例对象

* JavaScript 原生提供Error构造函数，所有抛出的错误都是这个构造函数的实例。JavaScript 语言标准只提到，Error实例对象必须有message属性，表示出错时的提示信息，没有提到其他属性。大多数 JavaScript 引擎，对Error实例还提供name和stack属性，分别表示错误的名称和错误的堆栈，但它们是非标准的，不是每种实现都有。
```javascript
var err = new Error('error message')
err.messgae // error message
```
* 在Error对象的基础上，JavaScript还定义了其他6个Error的派生对象：
1. SyntaxError 对象是解析代码时发生的语法错误。
2. ReferenceError对象是引用一个不存在的变量是发生的错误。
3. RangeError对象是一个值超出有效范围时发生的错误。
4. TypeError对象是变量或参数不是预期类型时发生的错误。调用对象不存在的方法，也会抛出TypeError错误，此时不存在的函数值是undefined而不是一个函数。
5. URIError对象是 URI 相关函数的参数不正确时抛出的错误，主要涉及encodeURI()、decodeURI()、encodeURIComponent()、decodeURIComponent()、escape()和unescape()这六个函数。
6. EvalError 对象。eval函数没有被正确执行时，会抛出EvalError错误。该错误类型已经不再使用了，只是为了保证与以前代码兼容，才继续保留。

#### throw 语句
* throw语句的作用是手动中断程序执行，抛出一个错误。
* throw可以抛出任何类型的值。也就是说，它的参数可以是任何值。
* 对于 JavaScript 引擎来说，遇到throw语句，程序就中止了。引擎会接收到throw抛出的信息，可能是一个错误实例，也可能是其他类型的值。

#### try...catch 结构
* try代码块抛出的错误，被catch代码块捕获后，程序会继续向下执行。

#### finally 代码块
* try...catch结构允许在最后添加一个finally代码块，表示不管是否出现错误，都必需在最后运行的语句。
* catch代码块之中，触发转入finally代码快的标志，不仅有return语句，还有throw语句。

```javascript
function f() {
  try {
    console.log(0);
    throw 'bug';
  } catch(e) {
    console.log(1);
    return true; // 这句原本会延迟到 finally 代码块结束再执行
    console.log(2); // 不会运行
  } finally {
    console.log(3);
    return false; // 这句会覆盖掉前面那句 return
    console.log(4); // 不会运行
  }

  console.log(5); // 不会运行
}

var result = f();
```
上述代码的执行结果为：0 1 3，result的值为false。

## 编程风格
* 下面三种情况，语法规定本来就不需要在结尾添加分号：
1.  for 和 while循环
2. 分支语句：if, switch, try
3. 函数的声明语句   

```javascript
for (;;){
}  //no need

while(){
} //no need

if (true) {
} //no need

switch () {
} //no need

try {
} catch {
} //no need

function f() {
} //no need
```

上面几种语句如果使用分号，并不会出错，解释引擎会把分号解释为空语句。
* do..while 循环需要分号，函数表达式仍然需要分号。   

```javascript
do {
  a--;
} while(a > 0); // 分号不能省略

var f = function f() {
}; // 分号不能省略 
```
