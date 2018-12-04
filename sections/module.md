# 模块

* [`[Basic]` 模块机制](/sections/module.md#模块机制)
* [`[Basic]` 热更新](/sections/module.md#热更新)
* [`[Basic]` 上下文](/sections/module.md#上下文)
* [`[Basic]` 包管理](/sections/module.md#包管理)

## 模块机制

### CommonJS规范

在 Node.js 模块系统中，每个文件都被视为独立的模块，有自己独立的作用域。

Node.js 模块系统遵循的是CommonJS规范。CommonJS规范加载模块是同步加载，只有加载完才能执行后续操作。
CommonJS模块的加载机制是，输入的是被输出的**值的拷贝**。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。

```js
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
```

```js
// main.js
var counter = require('./lib').counter;
var incCounter = require('./lib').incCounter;

console.log(counter);  // 3
incCounter();
console.log(counter); // 3
```

### 模块分类

Node的模块分类:

1. 核心模块：大多以C/C++编写，编译成二进制文件。部分核心模块在Node启动时已加载到内存中。
2. 文件模块：开发者编写的的模块，运行时动态加载，速度慢于核心模块。文件模块中，又分为 3 类模块
	1. .js。通过 fs 模块同步读取 js 文件并编译执行。
	2. .node。通过 C/C++ 进行编写的 Addon。通过 dlopen 方法进行加载。
	3. .json。读取文件，调用 JSON.parse 解析加载。

### 模块引入

在Node中引入模块的一般步骤：

1. 路径分析
2. 文件定位
3. 编译执行

`require`命令是CommonJS规范之中，用来加载其他模块的命令。它其实不是一个全局命令，而是指向当前模块的`module.require`命令，而后者又调用Node的内部命令`Module._load`。

```js
Module._load = function(request, parent, isMain) {
  // 1. 检查 Module._cache，是否缓存之中有指定模块
  // 2. 如果缓存之中没有，就创建一个新的Module实例
  // 3. 将它保存到缓存
  // 4. 使用 module.load() 加载指定的模块文件，
  //    读取文件内容之后，使用 module.compile() 执行文件代码
  // 5. 如果加载/解析过程报错，就从缓存删除该模块
  // 6. 返回该模块的 module.exports
};
```
`module.compile()`逻辑：

```js
Module.prototype._compile = function(content, filename) {
  // 1. 生成一个require函数，指向module.require
  // 2. 加载其他辅助方法到require
  // 3. 将文件内容放到一个函数之中，该函数可调用 require
  // 4. 执行该函数
};
```
一旦require函数准备完毕，整个所要加载的脚本内容，就被放到一个新的函数之中，这样可以避免污染全局环境。

```js
(function (exports, require, module, __filename, __dirname) {
  // YOUR CODE INJECTED HERE!
});
```

![](../assets/module-require.jpg)

> 参考：<br/>
> https://www.infoq.cn/article/nodejs-module-mechanism<br/>
> http://nodejs.cn/api/modules.html#modules_modules<br/>
> http://javascript.ruanyifeng.com/nodejs/module.html

## 热更新

热更新就是不重启程序的情况下，通过替换模块达到更新程序的过程。

在node中，require模块，若模块已经存在缓存中，则直接返回缓存中的模块。
可通过`require.cache`查看已缓存的模块。

```js
// c.js
console.log(require.cache)

// 打印结果

{ 'D:\\code\\c.js':
   Module {
     id: '.',
     exports: {},
     parent: null,
     filename: 'D:\\code\\c.js',
     loaded: false,
     children: [],
     paths: [ 'D:\\code\\node_modules', 'D:\\node_modules' ] } }
```

一般的热更新思路是监听修改的文件，然后在`require.cache`中删除这个文件的缓存，最后就是重新`require`这个文件。

```js
// a.js
module.exports = function () {
  const a = 18;
  console.log("the number is ", a)
}
```

```js
// b.js
var fs = require('fs');
var a = require('./a.js');

function cleanCache(module){
    var path = require.resolve(module);
    require.cache[path] = null;
}
b();
fs.watch(require.resolve('./a'),function(){
    console.log('change')
    cleanCache(require.resolve('./a'));
    try{
        a = require('./a');
        a();
    }catch(ex){
        console.log('module update failed');
    }
});
```

如果存在`A`引用`B`，`B`引用`C`，当只删除`B`的`require.cache`缓存，重新读取`B`中对`C`的引用数据，`C`返回的是缓冲中的数据，不会重新读取。

至于浏览器方面的热更新，简单的可以使用同样的思路配合websocket发送更新信息到浏览器实现立刻刷新。
或是使用Webpack HMR，可以做到保存状态不刷新的热替换，当中原理有点复杂，可以参考：[Webpack HMR 原理解析](https://zhuanlan.zhihu.com/p/30669007)

## 上下文

一个文件就是一个模块，模块包裹在函数里执行，有自己独立的作用域。
可以通过`global`定义全局变量。

```js
globalVal = 1

// ========

'use strict';
globalVar = 1 // 报错
global.globalVar = 1 // 正常
```
一般情况下不会污染全局变量，但如果有未经定义的全局变量，可能会产生污染。使用`use strict`严格模式会抛出错误，从而避免这个问题。

> 参考：https://www.zhihu.com/question/57375179/answer/152633354

## 包管理

### 锁版本

在开发中锁定模块版本，可以保证产品的稳定性与开发环境的一致性。

锁定版本的模块在`package.json`中写明版本号，不添加其他标记符号。

```js
"dependencies": {
    "vue": "2.2.0"
}
```
如果添加符号`~`或者`^`，代表模块安装更新的版本范围。

 - `~x.y.z`: 匹配大于 `x.y.z` 的 `z` 的最新版
 - `^x.y.z`: 匹配大于 `x.y.z` 的 `y.z` 的最新版
 - `*`: 匹配任何的依赖包

```js
~1.2.3  等价于 >=1.2.3 <1.(2+1).0 := >=1.2.3 <1.3.0
~1.2    等价于 >=1.2.0 <1.(2+1).0 := >=1.2.0 <1.3.0 (Same as 1.2.x)
~1      等价于 >=1.0.0 <(1+1).0.0 := >=1.0.0 <2.0.0 (Same as 1.x)
~0.2.3  等价于 >=0.2.3 <0.(2+1).0 := >=0.2.3 <0.3.0
~0.2    等价于 >=0.2.0 <0.(2+1).0 := >=0.2.0 <0.3.0 (Same as 0.2.x)
~0      等价于 >=0.0.0 <(0+1).0.0 := >=0.0.0 <1.0.0 (Same as 0.x)

^1.2.3  等价于 >=1.2.3 <2.0.0
^0.2.3  等价于 >=0.2.3 <0.3.0
^0.0.3  等价于 >=0.0.3 <0.0.4

*       等价于 >=0.0.0 (Any version satisfies)
1.x     等价于 >=1.0.0 <2.0.0 (Matching major version)
1.2.x   等价于 >=1.2.0 <1.3.0 (Matching major and minor versions)
```
> 参考：https://github.com/npm/node-semver#ranges

```js
yarn add package-name@1.2.3
```
可以通过`yarn`安装指定版本模块，模块版本就会被锁定。这个操作可能随着`yarn`的版本更迭而失效，但可手动去掉版本号前的符号达到锁版本。

`--save-exact/-E`参数强制npm在package.json中写死固定的版本号，而不使用如`~`，`^`这类的范围符号。

> 关于是否锁版本的意见：https://zhuanlan.zhihu.com/p/22934066