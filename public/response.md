# 响应(Response)

  Koa `Response` 对象是对 node 的 response 进一步抽象和封装，提供了日常 HTTP 服务器开发中一些有用的功能。

## API

### res.header

  Response header 对象。

### res.socket

  Request socket。

### res.status

  获取 response status。不同于 node 在默认情况下 `res.statusCode` 为200，`res.status` 并没有赋值。

### res.statusString

  Response status 字符串。

### res.status=

  通过 数字状态码或者不区分大小写的字符串来设置response status：

  - 100 "continue"
  - 101 "switching protocols"
  - 102 "processing"
  - 200 "ok"
  - 201 "created"
  - 202 "accepted"
  - 203 "non-authoritative information"
  - 204 "no content"
  - 205 "reset content"
  - 206 "partial content"
  - 207 "multi-status"
  - 300 "multiple choices"
  - 301 "moved permanently"
  - 302 "moved temporarily"
  - 303 "see other"
  - 304 "not modified"
  - 305 "use proxy"
  - 307 "temporary redirect"
  - 400 "bad request"
  - 401 "unauthorized"
  - 402 "payment required"
  - 403 "forbidden"
  - 404 "not found"
  - 405 "method not allowed"
  - 406 "not acceptable"
  - 407 "proxy authentication required"
  - 408 "request time-out"
  - 409 "conflict"
  - 410 "gone"
  - 411 "length required"
  - 412 "precondition failed"
  - 413 "request entity too large"
  - 414 "request-uri too large"
  - 415 "unsupported media type"
  - 416 "requested range not satisfiable"
  - 417 "expectation failed"
  - 418 "i'm a teapot"
  - 422 "unprocessable entity"
  - 423 "locked"
  - 424 "failed dependency"
  - 425 "unordered collection"
  - 426 "upgrade required"
  - 428 "precondition required"
  - 429 "too many requests"
  - 431 "request header fields too large"
  - 500 "internal server error"
  - 501 "not implemented"
  - 502 "bad gateway"
  - 503 "service unavailable"
  - 504 "gateway time-out"
  - 505 "http version not supported"
  - 506 "variant also negotiates"
  - 507 "insufficient storage"
  - 509 "bandwidth limit exceeded"
  - 510 "not extended"
  - 511 "network authentication required"

__注意__：不用担心记不住这些字符串，如果您设置错误，会有异常抛出，并列出该状态码表来帮助您进行更正。

### res.length=

  通过给定值设置 response Content-Length。

### res.length

  如果 Content-Length 作为数值存在，或者可以通过 `res.body` 来进行计算，则返回相应数值，否则返回 `undefined`。

### res.body

  获得 response body。

### res.body=

  设置 response body 为如下值：

  - `string` written
  - `Buffer` written
  - `Stream` piped
  - `Object` json-stringified
  - `null` no content response


  如果 `res.status` 没有赋值，Koa会自动设置为 `200` 或 `204`。

#### String

  Content-Type 默认为 text/html 或者 text/plain，两种默认 charset 均为 utf-8。 Content-Length 同时会被设置。

#### Buffer

  Content-Type 默认为 application/octet-stream，Content-Length同时被设置。

#### Stream

  Content-Type 默认为 application/octet-stream。

#### Object

  Content-Type 默认为 application/json。

### res.get(field)

  获取 response header 中字段值，field 不区分大小写。

```js
var etag = this.get('ETag');
```

### res.set(field, value)

  设置 response header 字段 `field` 的值为 `value`。

```js
this.set('Cache-Control', 'no-cache');
```

### res.set(fields)

  使用对象同时设置 response header 中多个字段的值。

```js
this.set({
  'Etag': '1234',
  'Last-Modified': date
});
```

### res.remove(field)

  移除 response header 中字段 `filed`。

### res.type

  获取 response `Content-Type`，不包含像 "charset" 这样的参数。

```js
var ct = this.type;
// => "image/png"
```

### res.type=

  通过 mime 类型的字符串或者文件扩展名设置 response `Content-Type`

```js
this.type = 'text/plain; charset=utf-8';
this.type = 'image/png';
this.type = '.png';
this.type = 'png';
```

  注意：当可以根据 `res.type` 确定一个合适的 `charset` 时，`charset` 会自动被赋值。
  比如 `res.type = 'html'` 时，charset 将会默认设置为 "utf-8"。然而当完整定义为 `res.type = 'text/html'`时，charset 不会自动设置。

### res.redirect(url, [alt])

  执行 [302] 重定向到对应 `url`。

  字符串 "back" 是一个特殊参数，其提供了 Referrer 支持。当没有Referrer时，使用 `alt` 或者 `/` 代替。

```js
this.redirect('back');
this.redirect('back', '/index.html');
this.redirect('/login');
this.redirect('http://google.com');
```

  如果想要修改默认的 [302] 状态，直接在重定向之前或者之后执行即可。如果要修改 body，需要在重定向之前执行。

```js
this.status = 301;
this.redirect('/cart');
this.body = 'Redirecting to shopping cart';
```

### res.attachment([filename])

  设置 "attachment" 的 `Content-Disposition`，用于给客户端发送信号来提示下载。filename 为可选参数，用于指定下载文件名。

### res.headerSent

  检查 response header 是否已经发送，用于在发生错误时检查客户端是否被通知。

### res.lastModified

  如果存在 `Last-Modified`，则以 `Date` 的形式返回。

### res.lastModified=

  以 UTC 格式设置 `Last-Modified`。您可以使用 `Date` 或 date 字符串来进行设置。

```js
this.response.lastModified = new Date();
```

### res.etag=

  设置 包含 `"`s 的 ETag。注意没有对应的 `res.etag` 来获取其值。

```js
this.response.etag = crypto.createHash('md5').update(this.body).digest('hex');
```

### res.append(field, val)

  在 header 的 `field` 后面 追加 `val`。

### res.vary(field)

  相当于执行res.append('Vary', field)。



