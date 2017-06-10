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
上面的例子是通过javascript关键字function来定义函数,同样也可以通过内置的javascript函数构造器Function()来定义。
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
提升(Hoisting)是javascript默认将当前作用域提升到前面去的行为。
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
在javascript中用typeof操作符判断函数类型时返回"function"。
但是把javascript函数描述为一个对象更加准确。
javascript函数有属性和方法。
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
