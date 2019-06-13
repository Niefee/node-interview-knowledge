# [util](/sections/util.md)

* [`[Doc]` URL](/sections/util.md#url)
* [`[Doc]` Query Strings (查询字符串)](/sections/util.md#query-strings)
* [`[Doc]` Utilities (实用函数)](/sections/util.md#util-1)
* [`[Basic]` 正则表达式](/sections/util.md#正则表达式)

## URL

`url` 模块用于处理与解析 URL。

`url` 模块提供了两套 API 来处理 URL：一个是旧版本遗留的 API，一个是实现了 WHATWG 标准的新 API。推荐使用新标准 API。

WHATWG 的 URL 对象的属性：

```js
┌────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                              href                                              │
├──────────┬──┬─────────────────────┬────────────────────────┬───────────────────────────┬───────┤
│ protocol │  │        auth         │          host          │           path            │ hash  │
│          │  │                     ├─────────────────┬──────┼──────────┬────────────────┤       │
│          │  │                     │    hostname     │ port │ pathname │     search     │       │
│          │  │                     │                 │      │          ├─┬──────────────┤       │
│          │  │                     │                 │      │          │ │    query     │       │
"  https:   //    user   :   pass   @ sub.example.com : 8080   /p/a/t/h  ?  query=string   #hash "
│          │  │          │          │    hostname     │ port │          │                │       │
│          │  │          │          ├─────────────────┴──────┤          │                │       │
│ protocol │  │ username │ password │          host          │          │                │       │
├──────────┴──┼──────────┴──────────┼────────────────────────┤          │                │       │
│   origin    │                     │         origin         │ pathname │     search     │ hash  │
├─────────────┴─────────────────────┴────────────────────────┴──────────┴────────────────┴───────┤
│                                              href                                              │
└────────────────────────────────────────────────────────────────────────────────────────────────┘
```

使用：

```js
const { URL } = require('url');
const myURL = new URL('/foo', 'https://example.org/');
console.log(myURL)
````

输出：
```bash
URL {
  href: 'https://example.org/foo?bar=12#bur',
  origin: 'https://example.org',
  protocol: 'https:',
  username: '',
  password: '',
  host: 'example.org',
  hostname: 'example.org',
  port: '',
  pathname: '/foo',
  search: '?bar=12',
  searchParams: URLSearchParams { 'bar' => '12' },
  hash: '#bur' }
```
> 文档：http://nodejs.cn/api/url.html#url_class_url

`URLSearchParams` API接口提供对URL查询字符串部分的读写权限。

```js
const { URL, URLSearchParams } = require('url');

const myURL = new URL('https://example.org/?abc=123');
console.log(myURL.searchParams.get('abc'));
// 输出 123

myURL.searchParams.append('abc', 'xyz');
console.log(myURL.href);
// 输出 https://example.org/?abc=123&abc=xyz

myURL.searchParams.delete('abc');
myURL.searchParams.set('a', 'b');
console.log(myURL.href);
// 输出 https://example.org/?a=b

const newSearchParams = new URLSearchParams(myURL.searchParams);
// 上面的代码等同于
// const newSearchParams = new URLSearchParams(myURL.search);

newSearchParams.append('a', 'c');
console.log(myURL.href);
// 输出 https://example.org/?a=b
console.log(newSearchParams.toString());
// 输出 a=b&a=c

// newSearchParams.toString() 被隐式调用
myURL.search = newSearchParams;
console.log(myURL.href);
// 输出 https://example.org/?a=b&a=c
newSearchParams.delete('a');
console.log(myURL.href);
// 输出 https://example.org/?a=b&a=c
```
> 文档：http://nodejs.cn/api/url.html#url_class_urlsearchparams

## query strings

`querystring` 模块提供用于解析和格式化 URL 查询字符串的实用工具。
功能与`URLSearchParams`接近，两者都不支持深度结构对象。

方法 | 描述
-----|-----
querystring.escape(str) | 对给定字符串进行百分比字符编码
querystring.unescape(str) | 在给定的 str 上执行 URL 百分比编码字符的解码。
querystring.stringify(obj[, sep[, eq[, options]]]) | 通过迭代对象的自身属性从给定的 obj 生成 URL 查询字符串。
querystring.parse(str[, sep[, eq[, options]]]) | 将 URL 查询字符串 str 解析为键值对的集合。

> 相关文档：http://nodejs.cn/api/querystring.html#querystring_query_string

## util

`util` 模块提供多种函数工具。

> 具体列表可以查阅：http://nodejs.cn/api/util.html#util_util

## 正则表达式

正则表达式常用于匹配字符，或匹配位置。
当中包含的内容非常多，可以逐渐学习相关知识：

> 1.http://louiszhai.github.io/2016/06/13/regexp<br/>
> 2.https://juejin.im/post/5965943ff265da6c30653879<br/>
> 3.https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions