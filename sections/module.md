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

> 参考： http://javascript.ruanyifeng.com/nodejs/module.html

### 模块分类

Node的模块分类:

1. 核心模块：大多以C/C++编写，编译成二进制文件。部分核心模块在Node启动时已加载到内存中。
2. 文件模块：开发者编写的的模块，运行时动态加载，速度慢于核心模块。
3. 第三方模块：非Node提供的文件模块，可以通过`require`引入第三方模块。

### 模块引入

第一次加载某个模块时，Node会缓存该模块。以后再加载该模块，就直接从缓存取出该模块的`module.exports`属性。

在Node中引入模块的一般步骤：

1. 路径分析
2. 文件定位
3. 编译执行