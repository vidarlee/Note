## let

### 不存在变量提升
var命令会发生“变量提升”（hoisting）现象。即变量可以在声明前使用，值为undefined。这种现象多多少少有些奇怪，为了纠正这种现象，let命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。
```
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

### 暂时性死区
只要在块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。
```
var tmp = 123;

if (true) {
  tmp = 'abc'; //ReferenceError
  let tmp;
}
```
ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。
```
typeof x; // ReferenceError
let x;
```

### 块级作用域与函数声明
函数能不能在块级作用域声明呢？
ES5规定，函数只能在顶层作用域和函数作用域中声明，不能在块级作用域声明。
```
// 情况一
if (true) {
  function f() {}
}

// 情况二
try {
  function f() {}
} catch(e) {
  // ...
}
```
上面两种函数声明，根据ES5的规定都是非法的。但是浏览器并没有遵循这个规定，为了兼容以前的旧代码还是支持块级作用域中声明函数，因此上面的代码实际都能执行，不会报错。

ES6引入了块级作用域，明确允许在块级作用域之中声明函数。ES6规定，块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。
```
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
```
上面代码在ES5中运行，会得到"I am inside!",因为在if代码块中声明的函数f会被提升至函数头部，实际运行的代码如下：
```
// ES5 环境
function f() { console.log('I am outside!'); }

(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f();
}());
```
ES6就完全不一样了，理论上会得到"I am outside!"。因为块级作用域内声明的函数类似于let,对作用域之外没有影响。但是如果你真在ES6浏览器中运行上面的代码，会报错，为什么呢？
原来，如果改变了块级作用域内声明的函数的处理规则，显然会对老代码产生很大的影响。为了减轻因此产生的不兼容问题，ES6规定，浏览器的实现可以不遵守上面的规定，可以有自己的行为方式。
- 允许在块作用域内声明函数
- 函数声明类似于var，即会提升到全局作用域或者函数作用域的头部。
- 同时，函数声明还会提升到所在的块级作用域的头部。
以上规则只对ES6浏览器适用，其他环境的实现不用遵守，还是将块级作用域的函数声明当做let处理。

根据这三条规则，在浏览器的ES6环境中，块级作用域内声明的函数，行为类似于var声明的变量。
```
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```
上面的代码在符合ES6的浏览器中，都会报错，因为实际运行的是下面的代码。
```
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```
考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式，而不是函数声明语句。
```
// 函数声明语句
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```
另外，还有一个需要注意的地方。ES6 的块级作用域允许声明函数规则，只在使用大括号的情况下成立，如果没有使用大括号，就会报错。
```
// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错
'use strict';
if (true)
  function f() {}
```

### do表达式
为了获取块级作用域的返回值，可以在块级作用域之前加上do，让它变成do表达式。
```
let x = do {
  let t = f();
  t * t + 1;
};
```
上面的代码中x会得到整个块级作用域的返回值。

## const

**const**声明一个只读的常量。一旦声明，常量的值不能被改变。
const声明的常量不能改变值，这意味着const一旦声明就必须立即初始化，不能留在以后初始化。
```
const foo;
// SyntaxError: Missing initializer in const declaration
```
const的作用域和let一样，只在声明所在的块级作用域有效。
```
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

### 本质
const保证的其实不是变量的值不得改变。而是变量指向的那个内存地址不能改变。对于简单的数据类型(数字，字符串，布尔型)，值就保存在变量指向的那个内存地址，因此等同于常量。但是对于复合型的数据类型(主要是对象和数组)，变量指向的内存地址，保存的只是一个指针，const只能保证这个指针是固定的，至于它指向的数据结构是不是可变，就完全不能控制了。
因此将一个对象声明成常量，需要非常小心。
```
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```
下面是另一个例子：
```
const a = [];
a.push('Hello'); // 可执行
a.length = 0;    // 可执行
a = ['Dave'];    // 报错
```
如果真的要将对象冻结，应该使用Object.freeze方法：
```
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
上面代码中，常量foo指向一个冻结的对象，所以添加新属性不起作用，严格模式时还会报错。

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。
```
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```
