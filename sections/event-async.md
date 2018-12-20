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
> 翻译：https://zhuanlan.zhihu.com/p/34451546

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