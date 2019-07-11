# [安全](/sections/security.md)

* [`[Doc]` Crypto (加密)](/sections/security.md#crypto)
* [`[Doc]` TLS/SSL](/sections/security.md#tlsssl)
* [`[Doc]` HTTPS](/sections/security.md#https)
* [`[Point]` XSS](/sections/security.md#xss)
* [`[Point]` CSRF](/sections/security.md#csrf)
* [`[Point]` 中间人攻击](/sections/security.md#中间人攻击)
* [`[Point]` Sql/Nosql 注入](/sections/security.md#sqlnosql-注入)

## crypto

> `crypto` 模块提供了加密功能，包括对 OpenSSL 的哈希、HMAC、加密、解密、签名、以及验证功能的一整套封装。

```js
const crypto = require('crypto');

const hash = crypto.createHash('md5');

// 可任意多次调用update():
hash.update('Hello, world!');
hash.update('Hello, nodejs!');

console.log(hash.digest('hex')); // 7e1977739c748beac0c0fd14fd26a544
```
> 官方文档：http://nodejs.cn/api/crypto.html#crypto_crypto

> 相关使用说明：https://www.cnblogs.com/chyingp/p/nodejs-learning-crypto-theory.html

## TLS/SSL

SSL/TLS协议用于保证互联网的通信安全上。

> 相关基础介绍：
> http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html

## https

HTTPS是以安全为目标的通信协议，在HTTP下加入SSL协议，保证客户端和服务器之间的加密通信安全。

基础介绍：https://juejin.im/post/5c889918e51d45346459994d

详细讲解：https://wetest.qq.com/lab/view/110.html

Node.js的HTTPS模块的多个类似于HTTP模块，但一开始创建服务器的时候需要引入秘钥。

> 私钥以及证书生产方式：http://nodejs.cn/api/tls.html#tls_tls_ssl_concepts

```js
// curl -k https://localhost:8000/
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('./agent2-key.pem'),
  cert: fs.readFileSync('./agent2-cert.pem')
};

https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('hello world\n');
}).listen(8000);
```

## xss

### XSS 分类

类型	   |   存储区*	|   插入点*
-------|------------|----------
存储型 XSS	| 后端数据库	 | HTML
反射型 XSS	| URL	     | HTML
DOM 型 XSS	| 后端数据库/前端存储/URL	| 前端 JavaScript

反射型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

```js
/*
 * 目标网站
 */
const express = require('express');
const app = express();

app.use(express.static('./xss'));
app.get('/', function(req, res) {
  res.setHeader('X-XSS-Protection', 0); // 这个处理是为了关闭现代浏览器的xss防护
  res.send('早上好' + req.query['name']);
});

app.listen(3000);
```

```html
<body>
<a href="http://localhost:3000/?name='<script>alert(/xss/)</script>'">点我进入目标网站</a>
</body>
```
点击进入目标网站，也会发现`<script>alert(/xss/)</script>`被写入页面中，并执行。


存储型 XSS 的攻击步骤：

1. 攻击者将恶意代码提交到目标网站的数据库中。
2. 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

> 存储型 XSS 的恶意代码常存在数据库里，反射型 XSS 的恶意代码常存在 URL 里。

在一个多行文本输入框中输入以前内容：

```html
<div style="color: red">
  欢迎大家阅读我的博客，我是WaterMan，喜欢就关注一下呗！
</div>
<script>
  alert('哈哈，我窃取了你的token了，下次注意噢' + document.cookie);
</script>
```

如果后端直接保存，用户打开网站查看输入内容时，用户本地的cookie就会被读取，进而被操作。

DOM 型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL。
3. 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

> DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

页面代码：
```html
<body>
  <div id="content"></div>
  <input type="text" id="text" value="" style="width: 500px" />
  <input type="button" id="submit" value="write" />
</body>
<script>
  document.getElementById('submit').onclick = function() {
    var str = document.getElementById('text').value;
    document.getElementById('content').innerHTML =
      "<a href='" + str + "'>testLink</a>";
  };
</script>
```

用户输入： 

```
' onclick=alert(/xss/) // 
```
页面代码就变成了：

```html
<a href='' onclick=alert(/xss/)//' > testLink</a>
```

> 用一个单引号闭合掉 href 的第一个单引号，然后插入一个 onclik 事件，最后再用注释符注释掉第二个单引号。点击这个新生成的链接，脚本将被执行。这样一来我们就发起了一次 DOM Based XSS 攻击。

> 参考：<br/>
> 1.https://juejin.im/post/5d169a12f265da1bd424942e#heading-0<br/>
> 2.https://segmentfault.com/a/1190000016551188#articleHeader0<br/>
> 3.https://github.com/dwqs/blog/issues/68

## csrf

CSRF全程 Cross Site Request Forgery, 攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。

示例：

1. 假如一家银行用以运行转账操作的URL地址如下： `http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName`

2. 那么，一个恶意攻击者可以在另一个网站上放置如下代码： `<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">`

3. 如果有账户名为Alice的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失1000资金。

POST类的CSRF会使用表单提交，

```html
<form action="http://bank.example/withdraw" method=POST>
    <input type="hidden" name="account" value="xiaoming" />
    <input type="hidden" name="amount" value="10000" />
    <input type="hidden" name="for" value="hacker" />
</form>
<script> document.forms[0].submit(); </script> 
```

### 防范策略

CSRF的两个特点：

1. CSRF（通常）发生在第三方域名。
2. CSRF攻击者不能获取到Cookie等信息，只是使用。

防护策略： 

 - 阻止不明外域的访问
	 - 同源检测(Origin Header/Referer Header)
	 - Samesite Cookie
 - 提交时要求附加本域才能获取的信息
	 - CSRF Token
	 - 双重Cookie验证

> 参考：https://juejin.im/post/5bc009996fb9a05d0a055192

## 中间人攻击

中间人攻击（Man-in-the-middle attack，缩写：MITM）是指攻击者与通讯的两端分别创建独立的联系，并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制。(来自[维基百科的描述](https://zh.wikipedia.org/wiki/%E4%B8%AD%E9%97%B4%E4%BA%BA%E6%94%BB%E5%87%BB))

> 详细教程以及代码实现：https://github.com/wuchangming/https-mitm-proxy-handbook

## Sql/Nosql 注入

SQL 注入是通过在用户可控参数中注入SQL语法，破坏原有SQL结构，达到编写程序时意料之外结果的攻击行为。

登录表单：
```html
<form action="/login" method="POST">
	<p>Username: <input type="text" name="username" /></p>
	<p>Password: <input type="password" name="password" /></p>
	<p><input type="submit" value="登陆" /></p>
</form>
```

SQL查询：

```sql
username:=r.Form.Get("username")
password:=r.Form.Get("password")
sql:="SELECT * FROM user WHERE username='"+username+"' AND password='"+password+"'"
```

```text
myuser' or 'foo' = 'foo' --
```

SQL会变成如下：

```sql
SELECT * FROM user WHERE username='myuser' or 'foo' = 'foo' --'' AND password='xxx'
```

在SQL里面`--`是注释标记，所以查询语句会在此中断。这就让攻击者在不知道任何合法用户名和密码的情况下成功登录了。

预防方法：

1. 权限限制，严格限制Web应用的数据库的操作权限。
2. 日志处理，当数据库操作失败的时候，尽量不要将原始错误日志返回。
3. 数据校验，来自用户的数据（GET, POST, cookie 等）最好做到以下两种过滤校验：
	1. 检查输入的数据是否具有所期望的数据格式。
	2. 使用数据库特定的敏感字符转义函数把用户提交上来的非数字数据进行转义。

> 参考：<br>
> 1.https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/09.4.md<br>
> 2.https://juejin.im/post/5bd5b820e51d456f72531fa8

NoSQL数据库没有使用传统的SQL语法，仍有收到注入攻击的可能。

NoSQL注入分类的分类方式：
1. 按照语言的分类:PHP数组注入，js注入和mongo shell拼接注入等等
2. 按照攻击机制分类:重言式，联合查询，JavaScript注入等等

> 参考：<br>
> 1.https://www.anquanke.com/post/id/97211<br>
> 2.https://blog.szfszf.top/tech/nosql%e6%b3%a8%e5%85%a5%e6%80%bb%e7%bb%93mongodb/