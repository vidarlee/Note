## 数据类型
- 字符串(String)
- 数字(Number)
- 布尔(Boolean)
- 数组(Array)
- 对象(Object)
- 空(Null)
- 未定义(Undefined)

## JavaScript拥有动态类型
```
var x;
var x = 5;
var x = "John";
```

## 字符串
字符串可以是引号中的人以文本，可以使用单引号或者双引号：
```
var x = "what about slash? \s \. \\";
var y = 'what about slash? \s \. \\';
var xx = "'' can be used in \" \"";
var yy = '"" can be used in \'\'';
```

在console里定义并输出相应的值：
```
>x
// "what about slash? s . \"
>y
// "what about slash? s . \"
>x == y
// true
>xx
// "'' can be used in " ""
>yy
// """ can be used in ''"
```
因此可以看出转移符\在双引号""和单引号''中都有效。

## 数字
JavaScript中的数字可以带小数点也可以不带，并可以使用科学计数法。
```
var value1 = 34.00;
var value2 = 34;
value1 == value2
// true
var value3 = 12e5;  //1200000
var value4 = 12e-5; //0.00012
```

## 布尔型
```
var x = true;
var y = false;
```

## 数组
数组的3种定义方式：
```
var cars = new Array();
cars[0] = "Saas";
cars[1] = "Volvo"
cars[2] = "BMW"
```
或者(condensed array)

`var cars = new Array("Saas", "Volvo", "BMW");`

或者(literal array)

`var cars = ["Saas", "Volvo", "BMW"]`

## 对象
对象有花括号分隔。在括号内部，对象的属性以名称和值的形式(name :   value)来定义。属性用逗号分隔。

`var person = {firstname:"Bill", lastname:"Gates", id:5566}`

上面例子中的对象(person)拥有三个属性：firstname，lastname和id。
也可以分多行定义：
```
var person = {
  firstname:"Bill";
  lastname:"Gates";
  id:5566
};
```
对象属性有两种寻址方式：
```
firstname=person.firstname;
firstname=person["firstname"];
```

## undefined和null
undefined表示变量中不含有值。
可以通过将变量的值设置为null来清空变量。
```
cars = null;
person = null;

var a = undefined;
var b = null;

a == b
// true

if (!a)
    console.log('undefined is false');
// undefined is false

if (!b)
    console.log('null is false');
// null is false

```
总而言之，undefined和null用法几乎差不多。

那为什么在JavaScript中会存在两个值来表示一样的意思呢？
### 历史原因
这与JavaScript的历史有关。1995年JavaScript诞生时，最初像Java一样，只设置了null作为表示"无"的值。
根据C语言的传统，null被设计成可以自动转为0。
```
Number(null)
// 0

5 + null
// 5
```
但是，JavaScript的设计者Brendan Eich，觉得这样做还不够，有两个原因。
首先，null像在Java里一样，被当成一个对象。但是，JavaScript的数据类型分成原始类型（primitive）和合成类型（complex）两大类，Brendan Eich觉得表示"无"的值最好不是对象。
其次，JavaScript的最初版本没有包括错误处理机制，发生数据类型不匹配时，往往是自动转换类型或者默默地失败。Brendan Eich觉得，如果null自动转为0，很不容易发现错误。
因此，Brendan Eich又设计了一个undefined。

### 最初的设计
JavaScript的最初版本是这样区分的：null是一个表示"无"的对象，转为数值时为0；undefined是一个表示"无"的原始值，转为数值时为NaN。
```
Number(undefined)
// NaN

5 + undefined
// NaN
```
### 目前的用法
但是，上面这样的区分，在实践中很快就被证明不可行。目前，null和undefined基本是同义的，只有一些细微的差别。
**null表示"没有对象"，即该处不应该有值。**典型用法是：

1. 作为函数的参数，表示该函数的参数不是对象。
2. 作为对象原型链的终点。

```
Object.getPrototypeOf(Object.prototype)
// null
```
**undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义。**典型用法是：

1. 变量被声明了，但没有赋值时，就等于undefined。
2. 调用函数时，应该提供的参数没有提供，该参数等于undefined。
3. 对象没有赋值的属性，该属性的值为undefined。
4. 函数没有返回值时，默认返回undefined。

```
var i;
i // undefined

function f(x){console.log(x)}
f() // undefined

var  o = new Object();
o.p // undefined

var x = f();
x // undefined
```
