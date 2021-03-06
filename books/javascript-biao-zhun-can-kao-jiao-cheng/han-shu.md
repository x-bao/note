### 函数声明

function 命令

```js
function init () {
    console.log('foo');
}
```

函数表达式

```js
// 讲一个匿名函数赋值给一个变量
var foo = function () {
    console.log('bar');
};

// 带有函数名的函数赋值给一个变量，则函数名只在函数内部有效
var foo = function bar () {
    console.log(typeof bar); // function
}

bar  // Uncaught ReferenceError: bar1 is not defined
*好处*：1）函数体内部调用自身；2）方便排错，据说是除错工具显示函数调用栈是将显示函数名，而不提示这里是一个匿名函数
```

Function 构造函数

```js
// 构造函数最后一个参数会被当成函数体，其他的都是这个函数体的参数
var add = new Function(
    'x',
    'y',
    'return x + y';
)
 <==>
 function add (x, y) {
     return x + y;
 }
```

### 函数重复声明

同一个函数可以多次声明，后边的声明覆盖前面的

### 函数是第一等公民

Js 中将函数看做是一种值，与数值，字符串等地位相同，故称为函数为第一等公民

### 函数名提升

采用 \`function\` 命令声明函数时，整个函数会像变量申明一样，提升到代码头部

```js
f();
// function 命令声明
function f() {}
// good

f();
// 函数表达式方式声明
var f = function () {};
// "TypeError: f is not a function
<==>
var f;
f();
f = function () { // todo };
```

### 不能在条件语句中声明函数

```js
// 😒 函数名提升，无效
if (false) {
    function f () {}
}

try {
    function f () {}
} catch (e) {
    console.log(e);
}

// 😊 使用函数表达式方式声明可以到达目的
if (false) {
    var f = function () {};
}
```

### name & length & toString

```js
function f1 () {}
f1.name // f1
var f2 = function () {}
f2.name // f2
var f3 = function myName() {}
f3.name // myName

// length：函数的预期传入参数个数
function f (a, b, c) () 
f.length // 3

// toString：返回函数的源码
function f () {
    console.log('toString：返回函数的源码')
}
f.toString();
// "function f() {
// console.log('toString：返回函数的源码')
// }"
```

### 函数作用域

函数作用域：**变量**存在的范围

* 全局作用域：变量在整个程序中都存在，所有地方可读取
* 函数作用域：变量只在函数内部存在

* 全局变量：函数外部声明的变量，所有地方可读取

* 局部变量：函数内部声明，外部无法读取，作用域内覆盖同名全局变量

### 函数内部变量提升

```js
// 函数内部声明变量同全局作用域一样
function (x) {
    if (x > 100) {
        var tmp = x - 100;
    }
}
<==>
function (x) {
    var tmp;
    if (x > 100) {
        tmp = x - 100;
    }
}
```

### 函数本身的作用域

* 函数执行时所在的作用域，是定义时的作用域，而不是调用时的作用域
* 函数体内部声明的函数，作用域绑定在函数体内部

```js
// 函数 x 声明在函数 f 之外，它的作用域绑定在全局，内部变量 a 不会在函数 f 内部取值
var a = 1;
var x = function () {
    console.log(a);
};
function f () {
    var a = 2;
    x();
}

f(); // 1

var x = function () {
    console.log(a);
};
function y (f) {
    var a = 2;
    f();
}

y(x); // Uncaught SyntaxError: Unexpected identifier

// 内部声明的函数，作用域绑定在函数体内部
// 函数 bar 在函数 foo 内部声明，作用域绑定在函数 foo 内部，在函数 foo 外部取出函数 bar 执行时，变量 a 指向函数体 foo 内部的 a
function foo () {
    var a = 1;
    function bar() {
        console.log(a);
    }
    return bar;
}

var a = 2;
var f = foo();
f(); // 1
```

### 参数

* 连续相同的参数取最后出现的值
* arguments 读取传入函数内的所有参数，类型：object
* 值传递：原始类型（数值，字符串，布尔值）
* 引用传递：复合类型（数组，对象，其他函数），
  * 只修改传入参数的某个值影响原始值
  * 整体替换不影响原始值

### 闭包

* 定义在函数内部的函数
* 链式作用域：子对象会一级一级向上寻找父对象的变量，父对象的所有变量对于子对象是可见的，反之不成立
* 闭包的作用

  * 读取函数内部的变量

  * 让这些变量始终保持在内存中

  * 封装对象的私有属性 & 私有方法

```js
// 读取函数内部的变量
function f1 () {
    var n = 999;
    // 闭包
    function f2() {
        console.log(n);
    }
    return f2;
}

var res = f1();
res(); // 999

// 变量保持在内存中
function add (i) {
    return function() {
        console.log(++i);
    }
}

var a = add(1);
a(); // 2
a(); // 3

// 封对象的私有属性 & 私有方法
function Person(name) {
    var _age;
    function setAge(n) {
        _age = n;
    }
    function getAge() {
        return _age;
    }
    return {
        name,
        getAge,
        setAge
    };
}

var Jim = new Person("Jim");
Jim.setAge(28);
Jim.getAge(); // 28
```

### 立即执行函数表达式（IIFE）

* 采用 function 命令声明函数，如果 function 出现在行首，一律解释为语句
* 通常情况下，只对匿名函数使用 IIFE，目的：
  * 不必为函数命名，避免污染全局变量
  * IIFE 内部构成独立作用域，封装外部无法读取的私有变量

```js
// 合法的 IIFE
(function () {} ());
(function () {})();

var i = function () { return 1; } ();

!function () {} ();
~function () {} ();
-function () {} ();
+function () {} ();

new function () {};
// 最后的括号用于传参
new function () {} ();

(function () {
    var tmp = newData;
    foo(tmp);
    bar(tmp);
})
```

### eval 命令

作用：将字符串当做语句执行

#### 调用方式

直接调用：eval 所在作用域为当前作用域

间接调用：eval 所在作用域为全局作用域

```js
// 直接调用
eval('return 1');

var a = 1;
eval('a = 2')
<= 2

// 严格模式下，直接调用仍然是当前作用域
(function f() {
  'use strict';
  var foo = 1;
  eval('foo = 2');
  console.log(foo);  // 2
})()

// 间接调用
var a = 1;

function f() {
  var a = 2;
  var e = eval;
  e('console.log(a)');
}

f() // 1
```



