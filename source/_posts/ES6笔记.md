---
title: ES6笔记（一）
date: 2019-06-18
categories: 学习笔记
tags:
  - javascript
---

> 参考资料：[http://es6.ruanyifeng.com](http://es6.ruanyifeng.com)

## let 和 const 命令

#### let 命令

ES6 新增的 let 命令，用于声明变量。用法类似 var，但所声明的变量，只在 let 命令所在的代码块有效。

```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function() {
    console.log(i);
  };
}
a[6](); // 10
//变量i用var声明，在全局范围都有效，所以全局只有一个变量i

var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function() {
    console.log(i);
  };
}
a[6](); // 6
//当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量

//for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
```

<!--more-->

##### 不存在变量提升

var 命令会发生“变量提升”现象，即变量可以在声明之前使用，值为 undefined。let 命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

##### 暂时性死区

ES6 明确规定，如果区块中存在 let 和 const 命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}

//“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。
typeof x; // ReferenceError
let x;

//作为比较，如果一个变量根本没有被声明，使用typeof反而不会报错。
typeof undeclared_variable; // "undefined"
```

##### 不允许重复声明

let 不允许在相同作用域内，重复声明同一个变量。

#### 块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。
第一种场景，内层变量可能会覆盖外层变量。

```javascript
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}

f(); // undefined
//原因在于变量提升，导致内层的tmp变量覆盖了外层的tmp变量。
```

第二种场景，用来计数的循环变量泄露为全局变量。

```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
//上面代码中，变量i只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。
```

**ES6 的块级作用域**
let 实际上为 JavaScript 新增了块级作用域。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); //5
}
// 外层作用域无法读取内层作用域中的内部变量
```

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
// IIFE 写法
// 避免污染全局变量
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```

**块级作用域与函数声明**
ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。

```javascript
// 情况一
if (true) {
  function f() {}
}

// 情况二
try {
  function f() {}
} catch (e) {
  // ...
}

//这两种函数声明，根据ES5规定都是非法的。
//浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数，因此上面两种情况实际都能运行，不会报错。
```

ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。ES6 规定，块级作用域之中，函数声明语句的行为类似于 let，在块级作用域之外不可引用。

```javascript
//在ES5中，会输出"I'm inside!"，因为函数提升，在if中的函数会被提升到函数头部
//在ES6中，理论上会输出"I'm outside!"，但是实际上会报错Uncaught TypeError: f is not a function
//原因在于如果改变了块级作用域内声明的函数的处理规则，会对老代码产生很大影响。
//所以ES6规定，浏览器的实现可以不遵守上面的规则，有自己的行为方式：
//1.允许在块级作用域内声明函数。
//2.函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
//3.同时，函数声明还会提升到所在的块级作用域的头部。
function f() {
  console.log('I am outside!');
}

(function() {
  if (false) {
    // 重复声明一次函数f
    function f() {
      console.log('I am inside!');
    }
  }

  f();
})();

//因此，浏览器的 ES6 环境实际是这样的
function f() {
  console.log('I am outside!');
}
(function() {
  var f = undefined;
  if (false) {
    function f() {
      console.log('I am inside!');
    }
  }

  f();
})();
// Uncaught TypeError: f is not a function
```

考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。

ES6 的块级作用域必须有大括号，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。

```javascript
// 第一种写法，报错
if (true) let x = 1;

// 第二种写法，不报错
if (true) {
  let x = 1;
}
```

#### const 命令

const 声明一个只读的常量。一旦声明，常量的值就不能改变。
const 声明的变量不得改变值，这意味着，const 一旦声明变量，就必须立即初始化，不能留到以后赋值。
const 的作用域与 let 命令相同：只在声明所在的块级作用域内有效。
const 命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。
const 声明的常量，也与 let 一样不可重复声明。

**本质**
const 实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。
简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const 只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。

```javascript
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop; // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only

//另一个例子
const a = [];
a.push('Hello'); // 可执行
a.length = 0; // 可执行
a = ['Dave']; // 报错
```

如果真的想将对象冻结，应该使用 Object.freeze 方法。

ES6 声明变量的六种方法：var、function、let、const、class、import

**顶层对象的属性**

顶层对象，在浏览器环境指的是 window 对象，在 Node 指的是 global 对象。ES5 之中，顶层对象的属性与全局变量是等价的。
ES6 为了改变这一点，一方面规定，为了保持兼容性，var 命令和 function 命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let 命令、const 命令、class 命令声明的全局变量，不属于顶层对象的属性。也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。

---

## 变量的解构赋值

#### 数组的解构赋值

ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

**模式匹配:只要等号两边的模式相同，左边的变量就会被赋予对应的值。**

```javascript
//以前，为变量赋值，只能直接指定值。
let a = 1;
let b = 2;
let c = 3;

//ES6 允许写成下面这样。
let [a, b, c] = [1, 2, 3];
//上面代码表示，可以从数组中提取值，按照对应位置，对变量赋值。
```

如果解构不成功，变量的值就等于 undefined。

```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo; // 1
bar; // 2
baz; // 3

let [x, , y] = [1, 2, 3];
x; // 1
y; // 3

// 失败的
let [foo] = [];
let [bar, foo] = [1];
//foo: undefined
```

如果等号的右边不是数组（或者严格地说，不是可遍历的结构，参见《Iterator》一章），那么将会报错。

```javascript
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```

**默认值**

```javascript
let [foo = true] = [];
foo; // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'

//默认值可以引用解构赋值的其他变量，但该变量必须已经声明。
let [x = 1, y = x] = []; // x=1; y=1
let [x = 1, y = x] = [2]; // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = []; // ReferenceError: y is not defined
```

#### 对象的解构赋值

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

```javascript
let { bar, foo } = { foo: 'aaa', bar: 'bbb' };
foo; // "aaa"
bar; // "bbb"

let { baz } = { foo: 'aaa', bar: 'bbb' };
baz; // undefined

//对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。
// 例一
let { log, sin, cos } = Math;

// 例二
const { log } = console;
log('hello'); // hello
```

如果变量名与属性名不一致，必须写成下面这样。

```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz; // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f; // 'hello'
l; // 'world'

//这实际上说明，对象的解构赋值是下面形式的简写
let { foo: foo, bar: bar } = { foo: 'aaa', bar: 'bbb' };
```
