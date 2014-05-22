# Context(上下文)

  Koa Context 将 node 的 `request` 和 `response` 对象封装在一个单独的对象里面，其为编写 web 应用和 API 提供了很多有用的方法。

  这些操作在 HTTP 服务器开发中经常使用，因此其被添加在上下文这一层，而不是更高层框架中，因此将迫使中间件需要重新实现这些常用方法。

  `context` 在每个 request 请求中被创建，在中间件中作为接收器(receiver)来引用，或者通过 `this` 标识符来引用：

```js
app.use(function *(){
  this; // is the Context
  this.request; // is a koa Request
  this.response; // is a koa Response
});
```

  许多 context 的访问器和方法为了便于访问和调用，简单的委托给他们的 `ctx.request` 和 `ctx.response` 所对应的等价方法，
  比如说 `ctx.type` 和 `ctx.length` 代理了 `response` 对象中对应的方法，`ctx.path` 和 `ctx.method` 代理了 `request` 对象中对应的方法。

## API

  `Context` 详细的方法和访问器。

### ctx.req

  Node 的 `request` 对象。

### ctx.res

  Node 的 `response` 对象。

  Koa _不支持_ 直接调用底层 res 进行响应处理。请避免使用以下 node 属性：

- `res.statusCode`
- `res.writeHead()`
- `res.write()`
- `res.end()`

### ctx.request

  Koa 的 `Request` 对象。

### ctx.response

  Koa 的 `Response` 对象。

### ctx.app

  应用实例引用。

### ctx.cookies.get(name, [options])

  
  获得 cookie 中名为 `name` 的值，`options` 为可选参数：
  - 'signed': 如果为 true，表示请求时 cookie 需要进行签名。

注意：Koa 使用了 Express 的 [cookies](https://github.com/jed/cookies) 模块，options 参数只是简单地直接进行传递。

### ctx.cookies.set(name, value, [options])

  设置 cookie 中名为 `name` 的值，`options` 为可选参数：

 - `signed`: 是否要做签名
 - `expires`: cookie 有效期时间
 - `path`: cookie 的路径，默认为 `/'`
 - `domain`: cookie 的域
 - `secure`: false 表示 cookie 通过 HTTP 协议发送，true 表示 cookie 通过 HTTPS 发送。
 - `httpOnly`: true 表示 cookie 只能通过 HTTP 协议发送

注意：Koa 使用了 Express 的 [cookies](https://github.com/jed/cookies) 模块，options 参数只是简单地直接进行传递。

### ctx.throw(msg, [status])

  抛出包含 `.status` 属性的错误，默认为 `500`。该方法可以让 Koa 准确的响应处理状态。
  Koa支持以下组合：

```js
this.throw(403)
this.throw('name required', 400)
this.throw(400, 'name required')
this.throw('something exploded')
```

  `this.throw('name required', 400)` 等价于：

```js
var err = new Error('name required');
err.status = 400;
throw err;
```

  注意：这些用户级错误被标记为 `err.expose`，其意味着这些消息被准确描述为对客户端的响应，而并非使用在您不想泄露失败细节的场景中。

### ctx.respond

  为了避免使用 Koa 的内置响应处理功能，您可以直接赋值 `this.repond = false;`。如果您不想让 Koa 来帮助您处理 reponse，而是直接操作原生 `res` 对象，那么请使用这种方法。

  注意： 这种方式是不被 Koa 支持的。其可能会破坏 Koa 中间件和 Koa 本身的一些功能。其只作为一种 hack 的方式，并只对那些想要在 Koa 方法和中间件中使用传统 `fn(req, res)` 方法的人来说会带来便利。

## Request aliases

  以下访问器和别名与 [Request](#request) 等价：

  - `ctx.header`
  - `ctx.method`
  - `ctx.method=`
  - `ctx.url`
  - `ctx.url=`
  - `ctx.originalUrl`
  - `ctx.path`
  - `ctx.path=`
  - `ctx.query`
  - `ctx.query=`
  - `ctx.querystring`
  - `ctx.querystring=`
  - `ctx.host`
  - `ctx.hostname`
  - `ctx.fresh`
  - `ctx.stale`
  - `ctx.socket`
  - `ctx.protocol`
  - `ctx.secure`
  - `ctx.ip`
  - `ctx.ips`
  - `ctx.subdomains`
  - `ctx.is()`
  - `ctx.accepts()`
  - `ctx.acceptsEncodings()`
  - `ctx.acceptsCharsets()`
  - `ctx.acceptsLanguages()`
  - `ctx.get()`

## Response aliases

  以下访问器和别名与 [Response](#response) 等价：

  - `ctx.body`
  - `ctx.body=`
  - `ctx.status`
  - `ctx.status=`
  - `ctx.length=`
  - `ctx.length`
  - `ctx.type=`
  - `ctx.type`
  - `ctx.headerSent`
  - `ctx.redirect()`
  - `ctx.attachment()`
  - `ctx.set()`
  - `ctx.remove()`
  - `ctx.lastModified=`
  - `ctx.etag=`

