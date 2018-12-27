# [事件/异步](/sections/event-async.md)

* [`[Basic]` Promise](/sections/event-async.md#promise)
* [`[Doc]` Events (事件)](/sections/event-async.md#events)
* [`[Doc]` Timers (定时器)](/sections/event-async.md#timers)
* [`[Point]` 阻塞/异步](/sections/event-async.md#阻塞异步)
* [`[Point]` 并行/并发](/sections/event-async.md#并行并发)

## promise

`promise`基础用法可以建议优先阅读阮一峰的文章 

 - [ECMAScript 6 入门 - Promise 对象](http://es6.ruanyifeng.com/#docs/promise)
 - [JavaScript 教程 - Promise 对象](https://wangdoc.com/javascript/async/promise.html)

### Event Loop

浏览器的`Event loop`和Node的`Event loop`是两个概念，下面主要讲Node方面的`Event loop`。

Node使用libuv引擎实现事件循环，它的实现比浏览器更加复杂，而且会跟内核交互。

> 官方相关说明文章：https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/<br/>
> 翻译：https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/

Event loop 的操作顺序：
```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

node中的事件循环的顺序：

```
外部输入数据-->轮询阶段(poll)-->检查阶段(check)-->关闭事件回调阶段(close callback)-->定时器检测阶段(timer)-->I/O事件回调阶段(I/O callbacks)-->闲置阶段(idle, prepare)-->轮询阶段...
```
> 每个阶段都有一个先进先出的回调函数队列。只有一个阶段的回调函数队列清空了，该执行的回调函数都执行了，事件循环才会进入下一个阶段。

各阶段的含义：

**（1）timers**

这个是定时器阶段，处理`setTimeout()`和`setInterval()`的回调函数。进入这个阶段后，主线程会检查一下当前时间，是否满足定时器的条件。如果满足就执行回调函数，否则就离开这个阶段。

**（2）I/O callbacks**

**除了以下**操作的回调函数，其他的回调函数都在这个阶段执行。

```
 - setTimeout()和setInterval()的回调函数
 - setImmediate()的回调函数
 - 用于关闭请求的回调函数，比如socket.on('close', ...)
```

**（3）idle, prepare**

该阶段只供 libuv 内部调用，这里可以忽略。

**（4）Poll**

这个阶段是轮询时间，用于等待还未返回的 I/O 事件，比如服务器的回应、用户移动鼠标等等。

这个阶段的时间会比较长。如果没有其他异步任务要处理（比如到期的定时器），会一直停留在这个阶段，等待 I/O 请求返回结果。

**（5）check**

该阶段执行`setImmediate()`的回调函数。

**（6）close callbacks**

该阶段执行关闭请求的回调函数，比如`socket.on('close', ...)`。

----

`process.nextTick()`并不是event loop的一部分。相反的，`process.nextTick()`会把回调塞入nextTickQueue，nextTickQueue将在当前阶段操作完成后执行，不管目前处于event loop的哪个阶段。

---

> 参考：<br/>
> http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html<br/>
> https://github.com/creeperyang/blog/issues/26<br/>

### Macrotask 和 Microtask

**Microtask**：
 1. process.nextTick
 2. promises
 3. Object.observe(废弃API)
 4. MutationObserver(监听DOM change)

**Macrotask**：
 1. setTimeout
 2. setInterval
 3. setImmediate
 4. I/O
 5. UI rendering
 6. requestAnimationFrame

> 按照[WHATWG](https://html.spec.whatwg.org/multipage/webappapis.html#task-queue) 规范，每一次事件循环（one cycle of the event loop），只处理一个 (macro)task。待该 macrotask 完成后，所有的 microtask 会在同一次循环中处理。处理这些 microtask 时，还可以将更多的 microtask 入队，它们会一一执行，直到整个 microtask 队列处理完。

> 参考：https://github.com/ccforward/cc/issues/47

## timers

`timer`模块是一个全局的 API，调度函数不需要使用`require`引入。

相关API可以查看官网文档：http://nodejs.cn/api/timers.html

## 阻塞异步


`同步`或者`异步`是有关消息通信机制的概念。**同步机制**，是指发送方发送请求后，需要等待接收到接收方发回的响应后，才接着发送下一个请求；**异步机制**，和同步机制正好相反，在异步机制中，发送方发出一个请求后，不等待接收方响应这个请求，就继续发送下个请求。

`阻塞`和`非阻塞`用来描述进程处理调用的方式。**阻塞调用**方式是调用结果返回之前，当前线程从运行状态被挂起，一直等到调用结果返回之后在继续执行。在**非阻塞**方式中，如果调用结果不能马上返回当前线程也不会被挂起，而是立即返回执行下一个调用。

在Node.js中，`阻塞` 是指程序中，JavaScript 语句的执行，必须等待一个非 JavaScript(IO) 操作完成。这是因为当 `阻塞`发生时，事件循环无法继续运行JavaScript。

```js
const fs = require('fs');
const data = fs.readFileSync('/file.md'); // 在这里阻塞直到文件被读取
```

`阻塞`方法`同步`执行，`非阻塞`方法`异步`执行。一般不使用`同步非阻塞`与`异步阻塞`。

> 参考：<br/>
> https://nodejs.org/zh-cn/docs/guides/blocking-vs-non-blocking/#<br/>
> https://www.zhihu.com/question/19732473

## 并行并发

![并行 (Parallel) 与并发 (Concurrent)](../assets/con_and_par.jpg)

	并发 (Concurrent) = 2 队列对应 1 咖啡机.
	并行 (Parallel) = 2 队列对应 2 咖啡机.

**并发**，在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理机上运行，但任一个时刻点上只有一个程序在处理机上运行。

**并行**，操作系统中是指，一组程序按独立异步的速度执行，无论从微观还是宏观，程序都是一起执行的。

>**并发性**是指在一段时间内宏观上有多个程序在同时运行，但在单处理机系统中，每一时刻却仅能有一道程序执行，故微观上这些程序只能是分时地交替执行。<br/>
>倘若在计算机系统中有多个处理机，则这些可以并发执行的程序便可被分配到多个处理机上，实现**并行执行**，即利用每个处理机来处理一个可**并发执行**的程序，这样多个程序便可以同时执行。

> 参考：<br/>
> https://baike.baidu.com/item/%E5%B9%B6%E5%8F%91#4<br/>
> https://baike.baidu.com/item/%E5%B9%B6%E8%A1%8C/5806759#reference-[1]-8050484-wrap
> https://www.zhihu.com/question/33515481

### Cluster

nodejs是单进程单线程，在多核机器上无法充分利用性能。nodejs引入`cluster`模块，可以提供多进程编程力能。

`cluster`模块通过`cluster.fork()`方法创建多个进程实例，而`cluster.fork()`内部又是通过`child_process.fork()`来创建子进程的。

```js
cluster.fork() --> child_process.for()
```

示例：

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程。
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
  });
} else {
  // 工作进程可以共享任何 TCP 连接。
  // 在本例子中，共享的是一个 HTTP 服务器。
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('你好世界\n');
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

打印：

```
$ node server.js
主进程 3596 正在运行
工作进程 4324 已启动
工作进程 4520 已启动
工作进程 6056 已启动
工作进程 5644 已启动
```

> 参考：http://nodejs.cn/api/cluster.html