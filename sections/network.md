# [Network](/sections/network.md)

* [`[Doc]` Net (网络)](/sections/network.md#net)
* [`[Doc]` UDP/Datagram](/sections/network.md#udp)
* [`[Doc]` HTTP](/sections/network.md#http)
* [`[Doc]` DNS (域名服务器)](/sections/network.md#dns)
* [`[Doc]` ZLIB (压缩)](/sections/network.md#zlib)
* [`[Point]` RPC](/sections/network.md#rpc)

## net

`net` 模块用于创建基于流的 `TC` 或 `IPC` 的服务器（`net.createServer()`）与客户端（`net.createConnection()`）。
`http`模块功能继承或者依赖于`net`模块。`net`对应传输层，`http`对应应用层。

net模块组成：

 - net.Server: tcp/server, 服务端TCP监听来自客户端的请求，并使用TCP连接(socket向客户端发送数据；内部通过socket来实现与客户端的通信；
 - net.Socket: tcp/本地，客户端TCP连接到服务器，并与服务器交换数据；socket的node实现，实现了全双工的stream的接口；

### 服务端net.Server

```js
let net = require('net')
let PORT = 8081
let HOST = 'localhost'
/**
 * 1. 创建一个TCP服务器实例，调用listen函数开始监听指定端口；
 * 2. 传入net.createServer()的回调函数，作为connection事件的处理函数；
 * 3. 在每个connection事件中，该回调函数接收到的socket对象是唯一的；
 * 4. 该连接自动关联一个socket对象
 * */
let server = net.createServer((socket) => {
    console.log('connection:' + socket.remoteAddress, socket.remotePort)
    // 为这个socket实例添加一个“data”事件处理函数
    socket.on('data', (data) => {
        console.log('DATA' + socket.remoteAddress + ":" + data);
        socket.write('You said "'+ data +'"\r\n') // 向客户端回发该数据

        // 如果想浏览器收到数据并正确解释，需要添加响应头，可以如下发送：
        /*
        socket.write(`
			HTTP/1.1 200 ok
			Content-Length: 11

			hello,world
		`);*/
    })
    
    /**
     * 服务端收到客户端发出的关闭连接请求时，会触发end事件
     * 这个时候客户端没有真正的关闭，只是开始关闭；
     * 当真正的关闭的时候，会触发close事件；
     * */
    socket.on('end', () => {
        console.log('客户端关闭')

        // 调用了该方法，则所有的客户端关闭跟本服务器的连接后，将关闭服务器
        // server.unref();
    })
    
    // 客户端关闭事件
    socket.on('close', () => {
        console.log('CLOSED: ' + socket.remoteAddress + ' ' + socket.remotePort);
    })
    
    // 设置客户端超时时间，如果客户端一直不输入，超过这个时间，就认为超时了
    socket.setTimeout(3000)
    socket.on('timeout', () => {
        console.log('超时了')
        // 默认情况下，当可读流读到末尾的时候会关闭可写流
        socket.pipe(socket, {end: false})
    })
})

server.listen(PORT, HOST, () => {
    console.log('服务端的地址是：', server.address())
})

server.on('error', (err) => {
    console.log(err)
})
```

服务端也可以通过显式处理”connection”事件来建立TCP连接，只是写法不同，二者没有区别。

```js
let server = net.createServer()
server.listen(PORT,HOST)
server.on('connection', (socket) => {
	console.log('CONNECTED: ' + sock.remoteAddress +':'+ sock.remotePort);
})
server.on('close', () => {
	//关闭服务器，停止接收新的客户端的请求
	console.log( 'close事件：服务端关闭' );
})

server.on('error', (error) => {
	console.log( 'error事件：服务端异常：' + error.message );
})
```

### 客户端net.Socket

```js
let net = require('net')

//创建一个TCP客户端连接到刚创建的服务器上，该客户端向服务器发送一串消息，并在得到服务器的反馈后关闭连接。

var client = new net.Socket()
let PORT = 8081
let HOST = 'localhost'

client.connect(PORT, HOST, () => {
	console.log('connect to ' + HOST + ':' + PORT)
	client.write('I am happyGloria.') //建立连接后立即向服务器发送数据，服务器将收到这些数据
})

client.on('data', (data) => {
	console.log('DATA: ' + data)

	// 完全关闭连接，没有关闭会导致超时。
	client.destroy() 
})

client.on('close', function () {
	console.log('Connection closed')
})
// client.end() // 显式结束连接
```

## udp

`UDP`，即用户数据报协议，一种面向无连接的传输层协议。与`TCP`相比，占用资源更少，传输速度更快。
`UDP`包括单播、广播和组播，暂时只讨论单播。

**单播**是向一个单播地址发送UDP数据报时，数据报只能被指定的IP主机接收，同一子网下的其它主机都不会接收该数据报。

服务端：
```js
const dgram = require('dgram');
const server = dgram.createSocket('udp4');

server.on('error', (err) => {
  console.log(`服务器异常：\n${err.stack}`);
  server.close();
});

server.on('message', (msg, rinfo) => {
  console.log(`服务器接收到来自 ${rinfo.address}:${rinfo.port} 的 ${msg}`);
});

server.on('listening', () => {
  const address = server.address();
  console.log(`服务器监听 ${address.address}:${address.port}`);
});

server.bind(41234);
```
> 相关接口：http://nodejs.cn/api/dgram.html

客户端：

一个发送 UDP 包到localhost上的某个随机端口的例子：
```js
const dgram = require('dgram');
const message = Buffer.from('Some bytes');
const client = dgram.createSocket('udp4');
client.send(message, 41234, 'localhost', (err) => {
  client.close();
});
```

一个发送包含多个 buffer 的 UDP 包到 127.0.0.1 上的某个随机端口的例子：

```js
const dgram = require('dgram');
const buf1 = Buffer.from('Some ');
const buf2 = Buffer.from('bytes');
const client = dgram.createSocket('udp4');
client.send([buf1, buf2], 41234, (err) => {
  client.close();
});
```
>一般来说，发送多个 buffer 速度更快。


> 关于广播与组播的信息，可查看：<br/>
> https://www.cnblogs.com/zmxmumu/p/6222946.html<br/>
> http://www.ayjs.net/post/179.html?utm_source=tuicool&utm_medium=referral

## http

关于http协议，建议先了解一下：http://www.ruanyifeng.com/blog/2016/08/http.html

URL通用的格式：scheme://host[:port#]/path/…/[?query-string][#anchor]

名称   | 功能
------ | -----
scheme | 访问服务器以获取资源时要使用哪种协议，比如，http，https 和 FTP 等
host | HTTP服务器的 IP 地址或域名
port | HTTP 服务器的默认端口是 80，这种情况下端口号可以省略，如果使用了别的端口，必须指明，例如www.site.com：8080
path | 访问资源的路径
query-string | 发给 http 服务器的数据
anchor | 锚

> HTTP相关知识：https://juejin.im/post/5a0ce1d95188253e24708454

`http`模块是对`net`模块的进一步封装，包含多个类。

### http.Agent 类

`Agent` 负责管理 HTTP 客户端的连接持久性和重用，为 `http.request`, `http.get` 提供代理服务的。

创建`http.Agent`对面的参数：

 - keepAlive <boolean> 即使没有未完成的请求，也要保持套接字，这样它们就可以被用于将来的请求而无需重新建立 TCP 连接。默认值: false。
 - keepAliveMsecs <number> 当使用 keepAlive 选项时，指定用于 TCP Keep-Alive 数据包的初始延迟。当 keepAlive 选项为 false 或 undefined 时则忽略。默认值: 1000。
 - maxSockets <number> 每个主机允许的套接字的最大数量。默认值: Infinity。
 - maxFreeSockets <number> 在空闲状态下保持打开的套接字的最大数量。仅当 keepAlive 被设置为 true 时才相关。默认值: 256。
 - timeout <number> 套接字的超时时间，以毫秒为单位。这会在套接字被连接之后设置超时时间。

使用 keepAlive 代理，有效的减少了建立/销毁连接的开销，开发者可以对连接池进行动态管理。
当不再使用时最好 使用`destroy()` 显式销毁 Agent 实例，因为未使用的套接字会消耗操作系统资源。

> 相关接口以及属性：http://nodejs.cn/api/http.html#http_class_http_agent 

> 参考：https://www.zhihu.com/question/58996077

### http.ClientRequest 类

此对象实例由 `http.request()` 内部创建并返回。 求创建后，并不会立即发送请求，我们还可以继续访问请求头：`setHeader(name, value)`、`getHeader(name)` 和 `removeHeader(name)` API 进行修改。实际的请求头会与第一个数据块一起发送或当调用 `request.end()` 时发送。

```js
// 引入http模块
const http = require('http');

// 创建一个请求
let request = http.request(
  {
    protocol: 'http:',     // 请求的协议
    host: 'aicoder.com',   // 请求的host
    port: 80,              // 端口
    method: 'GET',         // GET请求
    timeout: 2000,         // 超时时间
    path: '/'              // 请求路径
  },
  res => {  // 连接成功后，接收到后台服务器返回的响应，回调函数就会被调用一次。
    // res => http.IncomingMessage : 是一个Readable Stream
    res.on('data', data => {
      console.log(data.toString('utf8')); // 打印返回的数据。
    });
  }
);

// 设置请求头部
request.setHeader('Cache-Control', 'max-age=0');

// 真正的发送请求
request.end();
```

> 相关事件及属性：https://cloud.tencent.com/developer/article/1098198

### http.IncomingMessage 类

IncomingMessage 对象由 `http.Server` 或 `http.ClientRequest` 创建，并分别作为第一个参数传给 `'request'` 和 `'response'` 事件。 它可用于访问响应状态、消息头、以及数据。

### http.Server 类

此类继承自 `et.Server` 并具有额外的事件。

> 具体参考：http://nodejs.cn/api/http.html#http_class_http_server

### http.ServerResponse 类

此对象由 HTTP 服务器在内部创建，而不是由用户创建。 它作为第二个参数传给 'request' 事件。

响应继承自流，并额外实现内容。

> http://nodejs.cn/api/http.html#http_class_http_serverresponse

http所有类的思维导图总结：https://segmentfault.com/a/1190000011417058#articleHeader4

### http.createServer

使用 `http.createServer()` 方法创建一个 web 服务器返回一个 `Server` 实例。

```js
const http = require('http');

const proxy = http.createServer((req, res) => {
  console.log(req.headers);
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('响应内容 - 123');
});
proxy.listen(9000);
```
> 本地浏览器请求 `localhost:9000` 即可查看返回内容。

## dns

dns 模块包含两类函数：

1、使用底层操作系统工具进行域名解析，且无需进行网络通信。 这类函数只有一个：`dns.lookup()`

```js
const dns = require('dns');

dns.lookup('iana.org', (err, address, family) => {
  console.log('IP 地址: %j 地址族: IPv%s', address, family);
});
// IP 地址: "192.0.43.8" 地址族: IPv4
```

2、第二类函数，连接到真实的 DNS 服务器进行域名解析，且始终使用网络进行 DNS 查询。 这类函数包含了 `dns` 模块中除 `dns.lookup()` 以外的所有函数。

```js
const dns = require('dns');

dns.resolve4('archive.org', (err, addresses) => {
  if (err) throw err;

  console.log(`IP 地址: ${JSON.stringify(addresses)}`);
});
```

> 更多方法属性：http://nodejs.cn/api/dns.html#dns_dns

> 饿了么内容：https://elemefe.github.io/node-interview/#/sections/zh-cn/network?id=dns

使用`reverse()`方法反向解析IP地址:

```js
const dns = require('dns');
dns.reverse('203.188.200.67', (err, domain) => {
    // 反向解析IP地址
    console.log(domain);
    // [ 'media-router-fp1.prod.media.vip.tp2.yahoo.com' ]
});
```

> 大多数情况下，无法通过IP地址反向查找域名或者主机名。

> 总结：https://segmentfault.com/a/1190000013424394

## zlib

`zlib`模块提供通过 `Gzip` 和 `Deflate/Inflate` 实现的压缩功能。

压缩或者解压数据流(例如一个文件)通过zlib流将源数据流传输到目标流中来完成。

```js
const gzip = zlib.createGzip();
const fs = require('fs');
const inp = fs.createReadStream('input.txt');
const out = fs.createWriteStream('input.txt.gz');

inp.pipe(gzip).pipe(out);
```

也可以数据压缩或解压：

```js
const input = 'abc123';
zlib.deflate(input, (err, buffer) => {
  if (!err) {
    console.log(buffer.toString('base64')); // eJwzNDJOTEoGAAU8Ab0=
  } else {
    // 错误处理
  }
});

const buffer = Buffer.from('eJwzNDJOTEoGAAU8Ab0=', 'base64');
zlib.unzip(buffer, (err, buffer) => {
  if (!err) {
    console.log(buffer.toString());
  } else {
    // 错误处理
  }
});
````

`zlib` 可以用来实现对 `HTTP` 中定义的 `gzip` 和 `deflate` 内容编码机制的支持。

服务器：
```js
const zlib = require('zlib');
const http = require('http');
const fs = require('fs');
http.createServer((request, response) => {
  const raw = fs.createReadStream('a.html');
  let acceptEncoding = request.headers['accept-encoding'];
  if (!acceptEncoding) {
    acceptEncoding = '';
  }

  if (/\bdeflate\b/.test(acceptEncoding)) {
    console.log('flate');
    response.writeHead(200, { 'Content-Encoding': 'deflate',});
    raw.pipe(zlib.createDeflate()).pipe(response);
  } else if (/\bgzip\b/.test(acceptEncoding)) {
    console.log('gzip');
    response.writeHead(200, { 'Content-Encoding': 'gzip' });
    raw.pipe(zlib.createGzip()).pipe(response);
  } else {
    response.writeHead(200, {});
    raw.pipe(response);
  }
}).listen(1337);
```

客户端访问：

```js
const zlib = require('zlib');
const http = require('http');
const fs = require('fs');
const request = http.get({ host: 'localhost',
                           path: '/',
                           port: 1337,
                           headers: { 'Accept-Encoding': 'gzip,deflate' } });
request.on('response', (response) => {
  switch (response.headers['content-encoding']) {
    // 或者, 只是使用 zlib.createUnzip() 方法去处理这两种情况
    case 'gzip':
      response.pipe(zlib.createGunzip()).pipe(process.stdout);
      break;
    case 'deflate':
      response.pipe(zlib.createInflate()).pipe(process.stdout);
      break;
    default:
      response.pipe(output);
      break;
  }
});
```

> `zlib` 编码成本会很高, 结果应该被缓存。实际中应该权衡速度/内存/压缩。

> 更多方法以及属性：http://nodejs.cn/api/zlib.html#zlib_zlib

## rpc

RPC是指远程过程调用，通过网络对不在同一内存空间的程序进行调试。

> RPC采用客户机/服务器模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。首先，客户机调用进程发送一个有进程参数的调用信息到服务进程，然后等待应答信息。在服务器端，进程保持睡眠状态直到调用信息到达为止。当一个调用信息到达，服务器获得进程参数，计算结果，发送答复信息，然后等待下一个调用信息，最后，客户端调用进程接收答复信息，获得进程结果，然后调用执行继续进行。 --- 来自百度百科 [远程过程调用协议](https://baike.baidu.com/item/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8%E5%8D%8F%E8%AE%AE/6893245?fromtitle=RPC&fromid=609861&fr=aladdin)


推荐使用优秀的库进行尝试，以下示例使用`jayson`。

server端：

```js
var jayson = require('jayson')
 
const server = jayson.server({
  add: function(args, callback) {
    console.log('args:', args);
    callback(null, args[0] + args[1]);
  }
});

server.http().listen(3300);
```

client端：

```js
const jayson = require('jayson');

const client = jayson.client.http({
  port: 3300
});

// invoke "add"
client.request('add', [1, 1], function(err, response) {
  if(err) throw err;
  console.log('response:', response.result); // 2
});
```

或者使用Postman发起`post`请求，参数如下：

```json
{
	"jsonrpc": "2.0",
    "id": "1",
    "method": "add",
	"params" : [2,4]
}
```