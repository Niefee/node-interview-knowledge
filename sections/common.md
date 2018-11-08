# JavaScript 基础问题

* [`[Basic]` 数据类型](/sections/common.md#数据类型)
* [`[Basic]` 作用域](/sections/common.md#作用域)
* [`[Basic]` 引用传递](/sections/common.md#引用传递)
* [`[Basic]` 内存释放](/sections/common.md#内存释放)
* [`[Basic]` ES6 新特性](/sections/common.md#es6-新特性)

## 数据类型

在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。
JavaScript包含七种数据类型，分别为`number`、`string`、`boolean`、`undefined`、`null`、`object`和`symbol`(ES6新增的)。

`number`、`string`、`boolean`、`symbol`合称为原始类型（primitive type）数据。
`object`是由原始类型数据组成，所以称为复合类型（complex type）数据。
`undefined`和`null`，一般将它们看成两个特殊值。

`object`又可以分成三个子类型: 

 - 狭义的对象（object）
 - 数组（array）
 - 函数（function）

### 判断数据类型

常用的三种方法：

 - `typeof`运算符
 - `instanceof`运算符
 - `Object.prototype.toString`方法

#### typeof

 ```js
typeof 123 // "number"
typeof NaN // "number"

typeof "abc" // "string"  

typeof true // "boolean"  

typeof function foo() {} // "function"  

typeof undefined // "undefined"  

typeof Symbol('abc') // "symbol"
```
> 基本类型与函数都能直接返回数据类型的字符串。

```js
typeof {a: 1} // "object" 
typeof [1, 2] // "object" 
typeof null // "object"
 ```

js 在底层存储变量的时候，会在变量的机器码的低位1-3位存储其类型信息

 - 000：对象
 - 010：浮点数
 - 100：字符串
 - 110：布尔
 - 1：整数

`null`所有机器码均为0，所以用`typeof`判断返回`object`。
`undefined`用 −2^30 整数来表示

#### instanceof

`instanceof`的原理是检查右边构造函数的prototype属性，是否在左边对象的原型链上。也就是判断一个实例是否是其父类型或者祖先类型的实例。

实现原理大概如下：

```js
function instance_of(L, R) {//L 表示左表达式，R 表示右表达式
 var O = R.prototype;// 取 R 的显示原型
 L = L.__proto__;// 取 L 的隐式原型
 while (true) { 
   if (L === null) 
     return false; 
   if (O === L)// 这里重点：当 O 严格等于 L 时，返回 true 
     return true; 
   L = L.__proto__; 
 } 
}
```
> 参考：https://www.ibm.com/developerworks/cn/web/1306_jiangjj_jsinstanceof/

示例：
```js
'abc' instanceof String; // false, 字符串不是一个实例
123 instanceof Number; // false
true instanceof Boolean; // false

// 数字、布尔值同样效果
(new String('abc')) instanceof String; // true
(new String('abc')) instanceof Object; // true
String instanceof Object; // true

let person = function () {};
let Amy = new person();
Amy instanceof person; // true

[1,2] instanceof Array; // true
[1,2] instanceof Object; // true
```

简单来说，`instanceof` 的作用是检测一个对象是否是另个对象 `new` 出来的实例。牵涉更深一层原理，可参考 [js中\_\_proto\_\_和prototype的区别和关系？](https://www.zhihu.com/question/34183746/answer/59043879)

> 参考： https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof

#### Object.prototype.toString

调用`Object.prototype.toString.call(arguments)`会以`[object xxx]`返回参数的数据类型，`xxx`为类型名称。

```js
Object.prototype.toString.call(123) // "[object Number]"

Object.prototype.toString.call('1bc') // "[object String]"

Object.prototype.toString.call({a:'a'}) // "[object Object]"

Object.prototype.toString.call([1,'a', true]) // "[object Array]"

Object.prototype.toString.call(true) // "[object Boolean]"

Object.prototype.toString.call(() => {}) // "[object Function]"

Object.prototype.toString.call(null) // "[object Null]"

Object.prototype.toString.call(undefined) // "[object Undefined]"

Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"

Object.prototype.toString.call(new Date) // "[object Date]"

Object.prototype.toString.call(/\w+/ig) // "[object RegExp]"

Object.prototype.toString.call(new Error) // "[object Error]"
```

### 作用域

`JavaScript`的作用域有三个：

 - 全局作用域
 - 局部作用域
 - 块级作用域

#### 全局作用域

在函数定义之外声明的变量是`全局变量`，它的值可在整个程序中访问和修改。

```js
// 全局变量
var name = "John";
 
// 全局函数
function sayhi () {
    return console.log("Hi!");
}
```

#### 局部作用域

在函数定义内声明的变量是`局部变量`。  

```js
function sayhi () {
    var saying = "Hi, John.";
    console.log(saying);
}
console.log(saying); // 将抛出异常
```
#### 块级作用域
ES6新增`let`和`const`命令来声明变量。它们所声明的变量，只在命令所在的代码块内有效，这个代码块称为`块级作用域`。

```js
let x = 1;
{
  let x = 2, y=3;
}
console.log(x); // 1
console.log(y); // 报错，在外层没有定义变量y
```

> `const`同理。

> 参考：
> 1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/block
> 2. https://msdn.microsoft.com/zh-cn/library/bzt2dkta(v=vs.94).aspx

#### 模块作用域

在`Node.js`中，一个文件可以称为一个模块。每个模块都有各自的作用域，跨模块无法访问。可以通过`require`方法引入这个文件去调用当中作用域的值。或者通过`global`标志变量为全局作用域。

```js
// 引入文件
const file = require('./1.js');

// 全局变量
global.web = "Nodejs";
```

> 参考： http://nodejs.cn/api/globals.html

## 引用传递

### 堆和栈的概念

内存中存储的区域分为`堆`和`栈`。

> 栈（stack）为自动分配的内存空间，大小固定，它由系统自动释放；
> 堆（heap）则是动态分配的内存，大小不定也不会自动释放。

```js
var word = 'hello';
str[0] = 'y';

console.log(str); // hello，基本数据类型值不可变。
```

存储在`堆`中的一般是简单的数据段，它们位置存储的就是变量的值。JavaScript中的原始类型数据就是存储在`堆`中。

复合类型数据存储在`栈`中，但会在在`堆`中存储一个指向`栈`的指针。

### 传值与传址

原始类型数据的赋值运算在底层的实现是，新开一段`栈内存`，然后赋值到新栈中。

```js
var num1 = 10;
var num2 = num1;

num1 += 5;

console.log(num1); // 15
console.log(num2); // 20
```

> 赋值后变量的运算不会影响到原变量。

复合类型数据的赋值运算，是`堆`地址的赋值。新变量的`栈`内存获取的是原变量的`堆`地址。

```js
var a = {age: 12};
var b = a;

a.name = 'John'

a.age // 12
b.age // 12
b.name // 'John'
```

> 参考：
>  1. https://juejin.im/post/59ac1c4ef265da248e75892b 
>  2. https://segmentfault.com/a/1190000008637489

## 内存释放

不管什么程序语言，内存生命周期基本是一致的：   

1. 分配你所需要的内存
2. 使用分配到的内存（读、写）
3. 不需要时将其释放\归还

在C语言中，可以使用`malloc`方法用来申请内存，使用完后，必须自己用`free`方法释放内存。
但大多数语言提供自动内存管理，这被称为"垃圾回收机制"（garbage collector）。

```js
var n = 123; // 给数值变量分配内存
var s = "azerty"; // 给字符串分配内存

// 两个对象被创建，一个作为另一个的属性被引用，另一个被分配给变量o
// 很显然，没有一个可以被垃圾收集
var o = { 
  a: {
    b:2
  }
}; 

var o2 = o; // o2变量是第二个对“这个对象”的引用

o = 1;      // 现在，“这个对象”的原始引用o被o2替换了

var oa = o2.a; // 引用“这个对象”的a属性
// 现在，“这个对象”有两个引用了，一个是o2，一个是oa

o2 = "yo"; // 最初的对象现在已经是零引用了
           // 他可以被垃圾回收了
           // 然而它的属性a的对象还在被oa引用，所以还不能回收

oa = null; // a属性的那个对象现在也是零引用了
           // 它可以被垃圾回收了
```
> 参考：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management