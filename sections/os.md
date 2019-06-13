# [OS](/sections/os.md)

* [`[Doc]` TTY](/sections/os.md#tty)
* [`[Doc]` OS (os模块)](/sections/os.md#os模块)
* [`[Doc]` Path](/sections/os.md#path)
* [`[Doc]` 命令行参数](/sections/os.md#命令行参数)
* [`[Basic]` 负载](/sections/os.md#负载)
* [`[Point]` CheckList](/sections/os.md#checklist)

## TTY

`tty`（文本终端） 模块提供 `tty.ReadStream` 和 `tty.WriteStream` 类。 在大多数情况下，没有必要或可能直接使用此模块。

当 Node.js 检测到它附加了文本终端（TTY）时，默认情况下，`process.stdin` 将被初始化为 `tty.ReadStream` 的一个实例，`process.stdout` 和 `process.stderr` 将被初始化为 `tty.WriteStream` 的实例。判断 Node.js 是否在 TTY 上下文中运行的首选方法是检查 `process.stdout.isTTY` 属性的值是否为 `true`：

```js
$ node -p -e "Boolean(process.stdout.isTKY)"
true
$ node -p -e "Boolean(process.stdout.isTTY)" | cat
false
```

在终端运行`ps`命令，

```text
  PID    PPID    PGID     WINPID   TTY         UID    STIME COMMAND
 7960   15888    7960      16720  pty0     1085445 14:41:22 /usr/bin/ps
17196       1   17196      17196  ?        1085445 14:41:15 /usr/bin/mintty
15888   17196   15888      15860  pty0     1085445 14:41:16 /usr/bin/bash
```

其中为 `?` 的是没有依赖 TTY 的进程, 即守护进程。

> 文档：http://nodejs.cn/api/tty.html

## os模块

`os` 模块提供了操作系统相关的实用方法。 使用方法如下：

属性 | 	描述
-----|-----
os.EOL	|  根据当前系统, 返回当前系统的 End Of Line
os.arch()	|  返回当前系统的 CPU 架构, 如 'x86' 和 'x64'
os.constants	|  返回系统常量
os.cpus()	|  返回 CPU 每个核的信息
os.endianness()	|  返回 CPU 字节序, 如果是大端字节序返回 BE, 小端字节序则 LE
os.freemem()	|  返回系统空闲内存的大小, 单位是字节
os.homedir()	|  返回当前用户的根目录
os.hostname()	|  返回当前系统的主机名
os.loadavg()	|  返回负载信息
os.networkInterfaces()	|  返回网卡信息 (类似 ifconfig)
os.platform()	|  返回编译时指定的平台信息, 如 win32, linux, 同 process.platform()
os.release()	返回操作系统的分发版本号
os.tmpdir()	|  返回系统默认的临时文件夹
os.totalmem()	|  返回总内存大小(同内存条大小)
os.type()	|  根据 [uname](https://en.wikipedia.org/wiki/Uname#Examples) 返回系统的名称
os.uptime()	|  返回系统的运行时间，单位是秒
os.userInfo([options])	|  返回当前用户信息

> 不同操作系统的换行符 (EOL) 有什么区别?

end of line (EOL) 同 newline, line ending, 以及 line break 通常由 line feed (LF, \n) 和 carriage return (CR, \r) 组成. 常见的情况:

符号	 | 系统
-----|----
LF	|  在 Unix 或 Unix 相容系统 (GNU/Linux, AIX, Xenix, Mac OS X, ...)、BeOS、Amiga、RISC OS
CR+LF	|  MS-DOS、微软视窗操作系统 (Microsoft Windows)、大部分非 Unix 的系统
CR	|  Apple II 家族, Mac OS 至版本9

> 如果不了解 EOL 跨系统的兼容情况, 那么在处理文件的行分割/行统计等情况时可能会被坑.

> 参考：https://elemefe.github.io/node-interview/#/sections/zh-cn/os?id=os-1

还有OS常量，包括信号常量 (Signal Constants)、POSIX 错误常量 (POSIX Error Constants)、Windows 错误常量 (Windows Specific Error Constants)、libuv 常量 (libuv Constants)。

具体查看：http://nodejs.cn/api/os.html#os_os_constants_1

## path

`path` 模块提供用于处理文件路径和目录路径的实用工具。 

`path.basename()` 方法返回 `path` 的最后一部分，

```js
path.basename('/foo/bar/baz/asdf/quux.html');
// 返回: 'quux.html'

path.basename('/foo/bar/baz/asdf/quux.html', '.html');
// 返回: 'quux'
```

`path.join()`方法用于连接路径。该方法的能区别当前系统的路径分隔符，Unix系统是`/`，Windows系统是`\`。

```js
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
// 返回: '/foo/bar/baz/asdf'

path.join('foo', {}, 'bar');
// 抛出 'TypeError: Path must be a string. Received {}'
```

`path.resolve()` 方法将路径或路径片段的序列解析为绝对路径。

```js
path.resolve('/foo/bar', './baz');
// 返回: '/foo/bar/baz'

path.resolve('/foo/bar', '/tmp/file/');
// 返回: '/tmp/file'

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif');
// 如果当前工作目录是 /home/myself/node，
// 则返回 '/home/myself/node/wwwroot/static_files/gif/image.gif'
```

`path.relative()` 方法根据当前工作目录返回 `from` 到 `to` 的相对路径。

```js
path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb');
// 返回: '..\\..\\impl\\bbb'
```

`path.parse()` 方法返回一个路径各部分的信息对象。

```js
const path = require('path');
path.parse('D:\code\blog\source\about\index.md');

// 返回:
// { root: 'D:',
//   dir: 'D:',
//   base: 'code\blogsourceaboutindex.md',
//   ext: '.md',
//   name: 'code\blogsourceaboutindex' }

```

> 其他方式可查看文档：http://nodejs.cn/api/path.html#path_path

## 命令行参数

Node.js 自带了各种命令行选项。 这些选项开放了内置的调试、执行脚本的多种方式、以及其他有用的运行时选项。

 - node [options] [V8 options] [script.js | -e "script" | -] [--] [arguments]

 - node inspect [script.js | -e "script" | <host>:<port>] …

 - node --v8-options

 - 执行时不带参数，会启动 REPL。

> 相关参数说明：<br/>
> 1.https://elemefe.github.io/node-interview/#/sections/zh-cn/os?id=options<br/>
> 2.http://nodejs.cn/api/cli.html#cli_v_version

## 负载

UNIX相关系统可以通过`os.loadavg()`查看1, 5, 15分钟平均负载。
在Windows上,其返回值总是[0, 0, 0]。

在 Node.js 中单个进程的 CPU 负载查看可使用 `pidusage` 模块.

## checklist

略。
