## 函数

### 函数声明
```
function myFunction(a, b) {
    return a * b;
}
```

### 函数表达式
```
var x = function (a, b) { return a * b;}
```
在将函数表达式存储到变量后，变量也可以作为函数来使用：
```
var x = function (a, b) { return a * b;}
var z = x(1, 2);
```
以上函数实际上是一个匿名函数(函数没有名称)。
函数存储在变量中，不需要函数名称，通常通过变量名来调用。

### Function() 构造函数
上面的例子是通过JavaScript关键字function来定义函数,同样也可以通过内置的JavaScript函数构造器Function()来定义。
```
var myFunction = new Function("a", "b", "return a * b");
var x = myFunction(1, 2);
```
等同于
```
var myFunction = function (a, b) {return a * b};
var x = myFunction(1, 2);
```

### 函数提升
提升(Hoisting)是JavaScript默认将当前作用域提升到前面去的行为。
提升(Hoisting)应用在变量的声明与函数的声明。
也就是函数可以在声明前调用：
```
myFunction(5);

function myFunction(a){
  return a * a;
}
```
但是表达式定义函数时无法提升，因此下面的写法会出错：
Uncaught TypeError: myFunction is not a function
```
myFunction(5)
var myFunction = function (a) { retrun a * a }
```
这样则没有问题：
```
myFunction(5)
function myFunction(a) { return a * a }
```

### 自调用函数
函数表达式可以“自调用”。
自动用表达式可以自动调用。
自调用表达式后面紧跟()。
不能自调用声明函数。
通过添加括号，来说明它是一个函数表达式：
```
(function () {
  var x = "Hello, world!";
}) ();
```
以上函数实际上是一个匿名自我调用函数（无函数名）。

### 函数可以作为值使用
```
function myFunction(a, b) {
    return a * b;
}

var x = myFunction(4, 3);
```
### 函数作为表达式使用
```
function myFunction(a, b) {
    return a * b;
}

var x = myFunction(4, 3) * 2;
```

### 函数是对象
在JavaScript中用typeof操作符判断函数类型时返回"function"。
但是把JavaScript函数描述为一个对象更加准确。
JavaScript函数有属性和方法。
argument.length属性返回函数调用过程中接收到的参数个数：
```
function myFunction(a, b) {
    return arguments.length;
}
```
而toString()方法则将函数作为一个字符串返回：
```
function myFunction(a, b) {
    return a * b;
}

var txt = myFunction.toString();
```

### 使用构造函数调用函数
如果函数调用前使用了new关键字，则是调用了构造函数。
这看起来就像创建了新的函数，但实际上JavaScript函数是重新创建的对象：
```
//构造函数
function myFunction(arg1, arg2) {
  this.firstname = arg1;
  this.lastname = arg2;
}

// This creates a new object
var x = new myFunction("John", "Doe");
x.firstname;          // John
```
构造函数中this关键字没有任何的值。
this的值在函数调用时实例化对象(new object)时创建。

### 作为函数方法调用函数
在JavaScript中函数是对象。JavaScript函数有它的属性和方法。
call()和apply()是预定义的函数方法。两个方法可用于调用函数，两个方法的第一个参数必须是对象本身。
```
function myFunction(a, b) {
  return a * b;
}
myObject = myFunction.call(myObject, 10, 2);  // 20

myArray = [10, 2];
myObject = myFunction.apply(myObject, myArray); // 20
```
两个方法都使用了对象本身作为第一个参数。两者的区别在于第二个参数：apply传入的是一个参数数组，也就是将多个参数组合成为一个数组传入，而call则作为call的参数传入(从第二个参数开始)。
在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。
在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

**this是JavaScript语言的一个关键字。**
它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。比如：
```
function test() {
  this.x = 1;
}
```
**随着函数使用场合的不同, this的值会发生变化。但是有一个总的原则，那就是this值的是，调用函数的那个对象。**

也就是

In JavaScript this always refers to the “owner” of the function **we're executing**, or rather, to the object that a function is a method of.

**精髓就是this是指执行this相关代码时的对象。**

关于this的使用，请参考以下例子并给出相应的输出：
```
// 1.
var name = "The Window";
var object = {
  name : "My Object",
  getNameFunc : function(){
    return function(){
      return this.name;
    };
  }
};
alert(object.getNameFunc()());

// 2.
var name = "The Window";
var object = {
  name : "My Object",
  getNameFunc : function(){
    var that = this;
    return function(){
      return that.name;
    };
  }
};
alert(object.getNameFunc()());

// 3.
var name = "The Window";
var object = {
　name : "My Object",
　getNameFunc : function(){
  return this.name;
  }
};
alert(object.getNameFunc());

// 4.
var name = "The Window";
var object = {
  name : "My Object",
  getNameFunc : function(){
    return function(){
      return name;
    };
  }
};
alert(object.getNameFunc()());

// 5.
var name = "The Window";
var object = {
  name : "My Object",
  getNameFunc : function(){
    var _this = this;
    return function() {
      return _this.name +"__"+ this.name;
    };
  }
};
alert(object.getNameFunc()());
```

输出：
```
// 1. The Window
// 2. My Object
// 3. My Object
// 4. The Window
// 5. My Object__The Window
```
如果理解了this关键字指代的是在this语句被执行时的那个对象，那么就好理解上面的输出了。为了更好地理解，可以参考下面两个例子，这两个例子与上面的1，2是一样。
```
var name = "The Window";
var object = {
  name : "My Object",
  getNameFunc : function(){
    return function(){
      return this.name;	//这个this是有上下文的限制的
    };
  }
};
var tmp = object.getNameFunc(); //此时没有执行this.name
var name = tmp();//这个时候才执行，这时候的this上下文为全局
alert(name);

var name = "The Window";
var object = {
  name : "My Object",
  getNameFunc : function(){
    var that = this;
    return function(){
      return that.name;
    };
  }
};
var tmp = object.getNameFunc();	//这个时候执行了that = this，这里的this上下文是object,所以that指的是object
var name = tmp();	//这个时候执行了that.name
alert(name);
```
