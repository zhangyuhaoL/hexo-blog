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

**默认值**

默认值生效的条件是，对象的属性值严格等于 undefined。

```javascript
var { x = 3 } = { x: undefined };
x; // 3

var { x = 3 } = { x: null };
x; // null
```

**注意点**
（1）如果要将一个已经声明的变量用于解构赋值，必须非常小心。

```javascript
// 错误的写法
let x;
{x} = {x: 1};
// SyntaxError: syntax error

//上面代码的写法会报错，因为 JavaScript 引擎会将{x}理解成一个代码块，从而发生语法错误。只有不将大括号写在行首，避免 JavaScript 将其解释为代码块，才能解决这个问题。
// 正确的写法
let x;
({x} = {x: 1});
```

（2）解构赋值允许等号左边的模式之中，不放置任何变量名。因此，可以写出非常古怪的赋值表达式。

```javascript
({} = [true, false]);
({} = 'abc');
({} = []);

//虽然毫无意义，但是语法是合法的，可以执行。
```

（3）由于数组本质是特殊的对象，因此可以对数组进行对象属性的解构。

```javascript
let arr = [1, 2, 3];
let { 0: first, [arr.length - 1]: last } = arr;
first; // 1
last; // 3
```

#### 字符串的解构赋值

字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

```javascript
const [a, b, c, d, e] = 'hello';
a; // "h"
b; // "e"
c; // "l"
d; // "l"
e; // "o"

//类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值。
let { length: len } = 'hello';
len; // 5
```

#### 数值和布尔值的解构赋值

解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

```javascript
let { toString: s } = 123;
s === Number.prototype.toString; // true

let { toString: s } = true;
s === Boolean.prototype.toString; // true

//数值和布尔值的包装对象都有toString属性，因此变量s都能取到值。
```

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于 **undefined** 和 **null** 无法转为对象，所以对它们进行解构赋值，都会报错。

#### 函数参数的解构赋值

```javascript
//函数的参数也可以使用解构赋值。
function add([x, y]) {
  return x + y;
}

add([1, 2]); // 3

[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]

//函数参数的解构也可以使用默认值。
function move({ x = 0, y = 0 } = {}) {
  return [x, y];
}

move({ x: 3, y: 8 }); // [3, 8]
move({ x: 3 }); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]

//下面的写法会得到不一样的结果。
function move({ x, y } = { x: 0, y: 0 }) {
  return [x, y];
}

move({ x: 3, y: 8 }); // [3, 8]
move({ x: 3 }); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
//上面代码是为函数move的参数指定默认值，而不是为变量x和y指定默认值，所以会得到与前一种写法不同的结果。
```

#### 用途

```javascript
//（1）交换变量的值
let x = 1;
let y = 2;

[x, y] = [y, x];

//（2）从函数返回多个值
function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

function example() {
  return {
    foo: 1,
    bar: 2,
  };
}
let { foo, bar } = example();

//（3）函数参数的定义
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});

//（4）提取 JSON 数据
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]

//（5）函数参数的默认值
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

//（6）遍历 Map 结构
//任何部署了 Iterator 接口的对象，都可以用for...of循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

## 字符串的扩展

#### 模板字符串

模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```javascript
// 字符串中嵌入变量
let name = 'Bob',
  time = 'today';
`Hello ${name}, how are you ${time}?`;
```

## 字符串的新增方法

**实例方法：**

- includes()：返回布尔值，表示是否找到了参数字符串。
- startsWith()：返回布尔值，表示参数字符串是否在原字符串的头部。
- endsWith()：返回布尔值，表示参数字符串是否在原字符串的尾部。
  （这三个方法都支持第二个参数，表示开始搜索的位置。）
- repeat()：返回一个新字符串，表示将原字符串重复 n 次。（参数如果是小数，会被取整。）

**实例方法：**

- padStart()：用于头部补全；
- padEnd()：用于尾部补全；

```javascript
'x'.padStart(5, 'ab'); // 'ababx'
'x'.padStart(4, 'ab'); // 'abax'

'x'.padEnd(5, 'ab'); // 'xabab'
'x'.padEnd(4, 'ab'); // 'xaba'

//如果原字符串的长度，等于或大于最大长度，则字符串补全不生效，返回原字符串。
//如果用来补全的字符串与原字符串，两者的长度之和超过了最大长度，则会截去超出位数的补全字符串。
//如果省略第二个参数，默认使用空格补全长度。
```

**实例方法：**

- trimStart()：消除字符串头部的空格,不会修改原始字符串；
- trimEnd()：消除尾部的空格,不会修改原始字符串；

## 正则的扩展

#### RegExp 构造函数

ES5 中，RegExp 构造函数的参数有两种情况：

```javascript
//1.参数是字符串，这时第二个参数表示正则表达式的修饰符
var regex = new RegExp('xyz', 'i');
// 等价于
var regex = /xyz/i;

//2.参数是一个正则表示式，这时会返回一个原有正则表达式的拷贝。
var regex = new RegExp(/xyz/i);
// 等价于
var regex = /xyz/i;
//但是，ES5 不允许此时使用第二个参数添加修饰符，否则会报错。
var regex = new RegExp(/xyz/, 'i');
//Uncaught TypeError: Cannot supply flags when constructing one RegExp from another
```

ES6 改变了这种行为。如果 RegExp 构造函数第一个参数是一个正则对象，那么可以使用第二个参数指定修饰符。而且，返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。

```javascript
new RegExp(/abc/gi, 'i').flags;
// "i"
```

#### 字符串的正则方法

字符串对象共有 4 个方法，可以使用正则表达式：match()、replace()、search()和 split()。

ES6 将这 4 个方法，在语言内部全部调用 RegExp 的实例方法，从而做到所有与正则相关的方法，全都定义在 RegExp 对象上。

- String.prototype.match 调用 RegExp.prototype[Symbol.match]
- String.prototype.replace 调用 RegExp.prototype[Symbol.replace]
- String.prototype.search 调用 RegExp.prototype[Symbol.search]
- String.prototype.split 调用 RegExp.prototype[Symbol.split]

## 数值的扩展

#### 二进制和八进制表示法

二进制和八进制数值的新的写法，分别用前缀 0b（或 0B）和 0o（或 0O）表示。

#### Number.isFinite(), Number.isNaN()

Number.isFinite()用来检查一个数值是否为有限的（finite），即不是 Infinity。注意，如果参数类型不是数值，Number.isFinite 一律返回 false。

Number.isNaN()用来检查一个值是否为 NaN。

**区别：**
它们与传统的全局方法 isFinite()和 isNaN()的区别在于，传统方法先调用 Number()将非数值的值转为数值，再进行判断，而这两个新方法只对数值有效，Number.isFinite()对于非数值一律返回 false, Number.isNaN()只有对于 NaN 才返回 true，非 NaN 一律返回 false。

```javascript
isFinite(25); // true
isFinite('25'); // true
Number.isFinite(25); // true
Number.isFinite('25'); // false

isNaN(NaN); // true
isNaN('NaN'); // true
Number.isNaN(NaN); // true
Number.isNaN('NaN'); // false
Number.isNaN(1); // false
```

#### Number.parseInt(), Number.parseFloat()

ES6 将全局方法 parseInt()和 parseFloat()，移植到 Number 对象上面，行为完全保持不变。

```javascript
// ES5的写法
parseInt('12.34'); // 12
parseFloat('123.45#'); // 123.45

// ES6的写法
Number.parseInt('12.34'); // 12
Number.parseFloat('123.45#'); // 123.45

//这样做的目的，是逐步减少全局性方法，使得语言逐步模块化。
Number.parseInt === parseInt; // true
Number.parseFloat === parseFloat; // true
```

#### Number.isInteger()

Number.isInteger()用来判断一个数值是否为整数。

```javascript
Number.isInteger(25); // true
Number.isInteger(25.1); // false

//JavaScript 内部，整数和浮点数采用的是同样的储存方法，所以 25 和 25.0 被视为同一个值。
Number.isInteger(25); // true
Number.isInteger(25.0); // true
```

#### Math 对象的扩展

**Math.trunc():** 用于去除一个数的小数部分，返回整数部分。

```javascript
Math.trunc(4.1); // 4
Math.trunc(4.9); // 4
Math.trunc(-4.1); // -4
Math.trunc(-4.9); // -4
Math.trunc(-0.1234); // -0

//非数值，Math.trunc内部使用Number方法将其先转为数值。
Math.trunc('123.456'); // 123
Math.trunc(true); //1
Math.trunc(false); // 0
Math.trunc(null); // 0

//对于空值和无法截取整数的值，返回NaN。

//没有部署这个方法的环境，可以用下面的代码模拟。
Math.trunc =
  Math.trunc ||
  function(x) {
    return x < 0 ? Math.ceil(x) : Math.floor(x);
  };
```

**Math.sign():** 用来判断一个数到底是正数、负数、还是零。对于非数值，会先将其转换为数值。返回 5 种值：

- 参数为正数，返回+1；
- 参数为负数，返回-1；
- 参数为 0，返回 0；
- 参数为-0，返回-0;
- 其他值，返回 NaN。

没有部署这个方法的环境，可以用下面的代码模拟：

```javascript
Math.sign =
  Math.sign ||
  function(x) {
    x = +x; // convert to a number
    if (x === 0 || isNaN(x)) {
      return x;
    }
    return x > 0 ? 1 : -1;
  };
```

**Math.cbrt():** 用于计算一个数的立方根
对于没有部署这个方法的环境，可以用下面的代码模拟:

```javascript
Math.cbrt =
  Math.cbrt ||
  function(x) {
    var y = Math.pow(Math.abs(x), 1 / 3);
    return x < 0 ? -y : y;
  };
```

#### 指数运算符

ES2016 新增了一个指数运算符（\*\*）。

```javascript
2 ** 2; // 4
2 ** 3; // 8
```

这个运算符的一个特点是右结合，而不是常见的左结合。多个指数运算符连用时，是从最右边开始计算的。

// 相当于 2 ** (3 ** 2)
2 ** 3 ** 2;
// 512

## 函数的扩展

#### 函数参数的默认值

**基本用法**
ES6 之前，不能直接为函数的参数指定默认值，只能采用变通的方法。

```javascript
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello'); // Hello World
log('Hello', 'China'); // Hello China
log('Hello', ''); // Hello World

//这种写法的缺点在于，如果参数y赋值了，但是对应的布尔值为false，则该赋值不起作用。
//为了避免这个问题，通常需要先判断一下参数y是否被赋值，如果没有，再等于默认值。
if (typeof y === 'undefined') {
  y = 'World';
}
```

ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面。

```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello'); // Hello World
log('Hello', 'China'); // Hello China
log('Hello', ''); // Hello

//参数变量是默认声明的，所以不能用let或const再次声明。
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
```

**与解构赋值默认值结合使用**

```javascript
//只使用对象的解构赋值默认值，没有使用函数参数的默认值
function foo({ x, y = 5 }) {
  console.log(x, y);
}

foo({}); // undefined 5
foo({ x: 1 }); // 1 5
foo({ x: 1, y: 2 }); // 1 2
foo(); // TypeError: Cannot read property 'x' of undefined

//当参数不是一个对象时，变量x,y不会生成，就会报错。通过提供函数参数的默认值，就可以避免这种情况。
function foo({ x, y = 5 } = {}) {
  console.log(x, y);
}

foo(); // undefined 5
```

**函数的 length 属性**

指定了默认值以后，函数的 length 属性，将返回没有指定默认值的参数个数。

(function (a) {}).length // 1

(function (a = 5) {}).length // 0

(function (a, b, c = 5) {}).length // 2

如果设置了默认值的参数不是尾参数，那么 length 属性也不再计入后面的参数了。

(function (a = 0, b, c) {}).length // 0

(function (a, b = 1, c) {}).length // 1

**作用域**
一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的。

```javascript
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2); // 2

//函数f调用时，参数y = x形成一个单独的作用域。这个作用域里面，变量x本身没有定义，所以指向外层的全局变量x。函数调用时，函数体内部的局部变量x影响不到默认值变量x。
let x = 1;

function f(y = x) {
  let x = 2;
  console.log(y);
}

f(); // 1
//如果此时，全局变量x不存在，就会报错。
```

一个复杂的例子

```javascript
var x = 1;
function foo(
  x,
  y = function() {
    x = 2;
  }
) {
  var x = 3;
  y();
  console.log(x);
}

foo(); // 3
x; // 1
//函数foo的参数形成一个单独作用域。这个作用域里面，首先声明了变量x，然后声明了变量y，y的默认值是一个匿名函数。
//这个匿名函数内部的变量x，指向同一个作用域的第一个参数x。
//函数foo内部又声明了一个内部变量x，该变量与第一个参数x由于不是同一个作用域，所以不是同一个变量，因此执行y后，内部变量x和外部全局变量x的值都没变。

//如果将var x = 3的var去除，函数foo的内部变量x就指向第一个参数x，与匿名函数内部的x是一致的，所以最后输出的就是2，而外层的全局变量x依然不受影响。
var x = 1;
function foo(
  x,
  y = function() {
    x = 2;
  }
) {
  x = 3;
  y();
  console.log(x);
}

foo(); // 2
x; // 1
```

**应用**
利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误。

```javascript
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo();
// Error: Missing parameter
```

#### rest 参数

ES6 引入 rest 参数（形式为...变量名），用于获取函数的多余参数，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3); // 10

// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}
//arguments对象不是数组，而是一个类似数组的对象。所以为了使用数组的方法，必须使用Array.prototype.slice.call先将其转为数组。

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
//rest 参数就不存在这个问题，它就是一个真正的数组，数组特有的方法都可以使用。
```

**注意：** rest 参数之后不能再有其他参数（即只能是最后一个参数），否则会报错。
函数的 length 属性，不包括 rest 参数。

#### 箭头函数

ES6 允许使用“箭头”（=>）定义函数。

```javascript
var f = v => v;

//等同于
var f = function(v) {
  return v;
};
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```javascript
var f = () => 5;
// 等同于
var f = function() {
  return 5;
};

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};

//如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。
var sum = (num1, num2) => {
  return num1 + num2;
};
```

**注意点**

- 函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。(this 对象的指向是可变的，但是在箭头函数中，它是固定的。)
- 不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误。
- 不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
- 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

**不适用场合**
第一个场合是定义对象的方法，且该方法内部包括 this。

```javascript
const cat = {
  lives: 9,
  jumps: () => {
    this.lives--;
  },
};
```

cat.jumps()方法是一个箭头函数，这是错误的。调用 cat.jumps()时，如果是普通函数，该方法内部的 this 指向 cat；如果写成上面那样的箭头函数，使得 this 指向全局对象，因此不会得到预期结果。
这是因为对象不构成单独的作用域，导致 jumps 箭头函数定义时的作用域就是全局作用域。

第二个场合是需要动态 this 的时候，也不应使用箭头函数。

```javascript
var button = document.getElementById('press');
button.addEventListener('click', () => {
  this.classList.toggle('on');
});
```

上面代码运行时，点击按钮会报错，因为 button 的监听函数是一个箭头函数，导致里面的 this 就是全局对象。如果改成普通函数，this 就会动态指向被点击的按钮对象。

#### 尾调用优化

**什么是尾调用？**
尾调用（Tail Call）是函数式编程的一个重要概念，是指某个函数的最后一步是调用另一个函数。

```javascript
function f(x) {
  return g(x);
}

//以下都不属于尾调用
// 情况一
function f(x) {
  let y = g(x);
  return y;
}

// 情况二
function f(x) {
  return g(x) + 1;
}

// 情况三
function f(x) {
  g(x);
}
```

**尾调用优化**

函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数 A 的内部调用函数 B，那么在 A 的调用帧上方，还会形成一个 B 的调用帧。等到 B 运行结束，将结果返回到 A，B 的调用帧才会消失。如果函数 B 内部还调用函数 C，那就还有一个 C 的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）。

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3);

//如果函数g不是尾调用，函数f就需要保存内部变量m和n的值、g的调用位置等信息。
//但由于调用g之后，函数f就结束了，所以执行到最后一步，完全可以删除f(x)的调用帧，只保留g(3)的调用帧。
```

这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。

**注意:** 只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”。

```javascript
function addOne(a) {
  var one = 1;
  function inner(b) {
    return b + one;
  }
  return inner(a);
}
//上面的函数不会进行尾调用优化，因为内层函数inner用到了外层函数addOne的内部变量one。
```

**尾递归**
函数调用自身，称为递归。如果尾调用自身，就称为尾递归。
递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5); // 120
//计算n的阶乘，最多需要保存n个调用记录，复杂度 O(n) 。

//改写成尾递归，只保留一个调用记录，复杂度 O(1) 。
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1); // 120
------------------------------
//一个比较著名的例子，就是计算 Fibonacci 数列
//非尾递归的 Fibonacci 数列实现如下:
function Fibonacci (n) {
  if ( n <= 1 ) {return 1};

  return Fibonacci(n - 1) + Fibonacci(n - 2);
}

Fibonacci(10) // 89
Fibonacci(100) // 超时
Fibonacci(500) // 超时

//尾递归优化过的 Fibonacci 数列实现如下:
function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
  if( n <= 1 ) {return ac2};

  return Fibonacci2 (n - 1, ac2, ac1 + ac2);
}

Fibonacci2(100) // 573147844013817200000
Fibonacci2(1000) // 7.0330367711422765e+208
Fibonacci2(10000) // Infinity
```
