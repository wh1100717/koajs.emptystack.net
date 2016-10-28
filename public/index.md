# 安装

  Koa 目前需要 >=0.11.x版本的 node 环境。并需要在执行 node 的时候附带 --harmony 来引入 generators 。 如果您安装了较旧版本的 node ，您可以安装 [n](https://github.com/visionmedia/n) (node版本控制器)，来快速安装 0.11.x

```
$ npm install -g n
$ n 0.11.12
$ node --harmony my-koa-app.js
```


# 应用

  Koa 应用是一个包含一系列中间件 generator 函数的对象。
  这些中间件函数基于 request 请求以一个类似于栈的结构组成并依次执行。
  Koa 类似于其他中间件系统（比如 Ruby's Rack 、Connect 等），
  然而 Koa 的核心设计思路是为中间件层提供高级语法糖封装，以增强其互用性和健壮性，并使得编写中间件变得相当有趣。

  Koa 包含了像 content-negotiation（内容协商）、cache freshness（缓存刷新）、proxy support（代理支持）和 redirection（重定向）等常用任务方法。
  与提供庞大的函数支持不同，Koa只包含很小的一部分，因为Koa并不绑定任何中间件。

  任何教程都是从 hello world 开始的，Koa也不例外^_^  

```js
var koa = require('koa');
var app = koa();

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

## 中间件级联

  Koa 的中间件通过一种更加传统（您也许会很熟悉）的方式进行级联，摒弃了以往 node 频繁的回调函数造成的复杂代码逻辑。
  我们通过 generators 来实现“真正”的中间件。
  Connect 简单地将控制权交给一系列函数来处理，直到函数返回。
  与之不同，当执行到 `yield next` 语句时，Koa 暂停了该中间件，继续执行下一个符合请求的中间件('downstream')，然后控制权再逐级返回给上层中间件('upstream')。

  下面的例子在页面中返回 "Hello World"，然而当请求开始时，请求先经过 `x-response-time` 和 `logging` 中间件，并记录中间件执行起始时间。
  然后将控制权交给 `reponse` 中间件。当中间件运行到 `yield next` 时，函数挂起并将控制前交给下一个中间件。当没有中间件执行 `yield next` 时，程序栈会逆序唤起被挂起的中间件来执行接下来的代码。

```js
var koa = require('koa');
var app = koa();

// x-response-time

app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  this.set('X-Response-Time', ms + 'ms');
});

// logger

app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  console.log('%s %s - %s', this.method, this.url, ms);
});

// response

app.use(function *(){
  this.body = 'Hello World';
});

app.listen(3000);
```

## 配置

  应用配置是 `app` 实例属性，目前支持的配置项如下：

  - `app.name` 应用名称（可选项）
  - `app.env` 默认为 __NODE_ENV__ 或者 `development`
  - `app.proxy` 如果为 `true`，则解析 "Host" 的 header 域，并支持 `X-Forwarded-Host`
  - `app.subdomainOffset` 默认为2，表示 `.subdomains` 所忽略的字符偏移量。

## app.listen(...)

  Koa 应用并非是一个 1-to-1 表征关系的 HTTP 服务器。
  一个或多个Koa应用可以被挂载到一起组成一个包含单一 HTTP 服务器的大型应用群。

  如下为一个绑定3000端口的简单 Koa 应用，其创建并返回了一个 HTTP 服务器，为 `Server#listen()` 传递指定参数（参数的详细文档请查看[nodejs.org](http://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback)）。

```js
var koa = require('koa');
var app = koa();
app.listen(3000);
```

  `app.listen(...)` 实际上是以下代码的语法糖:

```js
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);
```

  这意味着您可以同时支持 HTTPS 和 HTTPS，或者在多个端口监听同一个应用。

```js
var http = require('http');
var koa = require('koa');
var app = koa();
http.createServer(app.callback()).listen(3000);
http.createServer(app.callback()).listen(3001);
```

## app.callback()

  返回一个适合 `http.createServer()` 方法的回调函数用来处理请求。
  您也可以使用这个回调函数将您的app挂载在 Connect/Express 应用上。

## app.use(function)

  为应用添加指定的中间件，详情请看 [Middleware](https://github.com/koajs/koa/wiki#middleware)

## app.keys=

  设置签名Cookie密钥，该密钥会被传递给 [KeyGrip](https://github.com/jed/keygrip)。

  当然，您也可以自己生成 `KeyGrip` 实例：

```js
app.keys = ['im a newer secret', 'i like turtle'];
app.keys = new KeyGrip(['im a newer secret', 'i like turtle'], 'sha256');
```

  在进行cookie签名时，只有设置 `signed` 为 `true` 的时候，才会使用密钥进行加密：

```js
this.cookies.set('name', 'tobi', { signed: true });
```

## 错误处理

  默认情况下Koa会将所有错误信息输出到 stderr，除非 __NODE\_ENV__ 是 "test"。为了实现自定义错误处理逻辑（比如 centralized logging），您可以添加 "error" 事件监听器。

```js
app.on('error', function(err){
  log.error('server error', err);
});
```

  如果错误发生在 请求/响应 环节，并且其不能够响应客户端时，`Contenxt` 实例也会被传递到 `error` 事件监听器的回调函数里。

```js
app.on('error', function(err, ctx){
  log.error('server error', err, ctx);
});
```

  当发生错误但仍能够响应客户端时（比如没有数据写到socket中），Koa会返回一个500错误(Internal Server Error)。

  无论哪种情况，Koa都会生成一个应用级别的错误信息，以便实现日志记录等目的。


