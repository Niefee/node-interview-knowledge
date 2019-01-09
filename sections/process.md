# [进程](/sections/process.md)

* [`[Doc]` Process (进程)](/sections/process.md#process)
* [`[Doc]` Child Processes (子进程)](/sections/process.md#child-process)
* [`[Doc]` Cluster (集群)](/sections/process.md#cluster)
* [`[Basic]` 进程间通信](/sections/process.md#进程间通信)
* [`[Basic]` 守护进程](/sections/process.md#守护进程)

## process

`process` 对象是一个全局变量，提供 Node.js 进程的有关信息以及控制进程。
官方文档：http://nodejs.cn/api/process.html

## 操作系统中的进程与线程

对于操作系统来说，一个任务就是一个`进程（Process）`，比如打开一个浏览器就是启动一个浏览器进程，打开一个记事本就启动了一个记事本进程，打开两个记事本就启动了两个记事本进程，打开一个Word就启动了一个Word进程。

有些进程还不止同时干一件事，比如Word，它可以同时进行打字、拼写检查、打印等事情。在一个进程内部，要同时干多件事，就需要同时运行多个“子任务”，我们把进程内的这些“子任务”称为`线程（Thread）`。

> 参考：[进程和线程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014319272686365ec7ceaeca33428c914edf8f70cca383000)