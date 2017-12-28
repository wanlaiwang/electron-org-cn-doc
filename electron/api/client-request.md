## 类: ClientRequest

> 发起HTTP/HTTPS请求.

线程：[主线程](../glossary.md#main-process)

`ClientRequest`实现了[Writable Stream](https://nodejs.org/api/stream.html#stream_writable_streams)接口, 因此是一个[EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter)类型.

### `new ClientRequest(options)`

* `options` (Object | String) -如果 `options` 是一个String类型, 它被解释为请求的URL. 如果它是一个Object类型, 那么它可以通过以下属性指定一个HTTP请求: 
  * `method` String (可选) - HTTP请求方法. 默认为GET方法.
  * `url` String (可选) - 请求的URL. 必须在指定了http或https的协议方案的独立表单中提供.
  * `session` Object (可选) - 与请求相关联的[`Session`](session.md)实例.
  * `partition` String (可选) - 与请求相关联的[`partition`](session.md)名称. 默认为空字符串. `session`选项优先于`partition`选项. 因此, 如果`session`是显式指定的, 则`partition`将被忽略.
  * `protocol` String (可选) - 在"scheme:"表单中的协议方案. 目前支持的值为'http:' 或者'https:'. 默认为'http:'.
  * `host` String (可选) - 作为连接提供的服务器主机,主机名和端口号'hostname:port'
  * `hostname` String (可选) - 服务器主机名.
  * `port` Integer (可选) - 服务器侦听的端口号.
  * `path` String (可选) - 请求URL的路径部分.
  * `redirect` String (可选) - 请求的重定向模式. 可选值为 `follow`, `error` 或 `manual`. 默认值为 `follow`. 当模式为`error`时, 重定向将被终止. 当模式为 `manual`时，表示延迟重定向直到调用了 [`request.followRedirect`](#requestfollowRedirect)。 在此模式中侦听 [`redirect`](#event-redirect)事件，以获得关于重定向请求的更多细节。

`options` 属性，如 `protocol`, `host`, `hostname`, `port` 和 `path`，在 [URL](https://nodejs.org/api/url.html) 模块中会严格遵循 Node.js 的模式

例如,我们可以创建与github.com相同的请求如下:

```JavaScript
const request = net.request({
  method: 'GET',
  protocol: 'https:',
  hostname: 'github.com',
  port: 443,
  path: '/'
})
```

### 实例事件

#### Event: 'response'

返回:

* `response` 收到的消息 - 表示HTTP响应消息的对象。

#### 事件: "login"

返回:

* `authInfo` Object 
  * `isProxy` Boolean
  * `scheme` String
  * `host` String
  * `port` Integer
  * `realm` String
* `callback` Function 
  * `username` String
  * `password` String

当身份验证代理请求用户认证时触发

用户证书会调用 `callback`方法:

* `username` String
* `password` String

```JavaScript
request.on('login', (authInfo, callback) => {
  callback('username', 'password')
})
```

提供空的凭证将取消请求，并在响应对象上报告一个身份验证错误:

```JavaScript
request.on('response', (response) => {
  console.log(`STATUS: ${response.statusCode}`);
  response.on('error', (error) => {
    console.log(`ERROR: ${JSON.stringify(error)}`)
  })
})
request.on('login', (authInfo, callback) => {
  callback()
})
```

#### Event: 'finish'

在 `request` 最终的 chunk 数据后写入 `request` 后触发

#### Event: 'abort'

当 `request`请求被中止时发出。如果`request` 请求已经关闭， `abort`中止事件将不会被触发。

#### Event: 'error'

返回:

* `error` Error -提供失败信息的错误对象。

当 `net`网络模块没有发出网络请求时会触发。 通常情况下，当 `request`请求对象发出一个 `error`错误事件时，一个 `close`关闭事件会随之发生，并且不会提供响应对象。

#### 事件：close

作为HTTP 的 request-response 中的最后一个事件发出。 `close`事件表明，在`request`或`response` 对象中不会发出更多的事件。

#### Event: 'redirect'

返回:

* `statusCode` Integer
* `method` String
* `redirectUrl` String
* `responseHeaders` Object

Emitted when there is redirection and the mode is `manual`. Calling [`request.followRedirect`](#requestfollowRedirect) will continue with the redirection.

### 实例属性

#### `request.chunkedEncoding`

A `Boolean` specifying whether the request will use HTTP chunked transfer encoding or not. Defaults to false. The property is readable and writable, however it can be set only before the first write operation as the HTTP headers are not yet put on the wire. Trying to set the `chunkedEncoding` property after the first write will throw an error.

Using chunked encoding is strongly recommended if you need to send a large request body as data will be streamed in small chunks instead of being internally buffered inside Electron process memory.

### 实例方法

#### `request.setHeader(name, value)`

* `name` String - An extra HTTP header name.
* `value` Object - An extra HTTP header value.

Adds an extra HTTP header. The header name will issued as it is without lowercasing. It can be called only before first write. Calling this method after the first write will throw an error. If the passed value is not a `String`, its `toString()` method will be called to obtain the final value.

#### `request.getHeader(name)`

* `name` String - Specify an extra header name.

Returns `Object` - The value of a previously set extra header name.

#### `request.removeHeader(name)`

* `name` String - Specify an extra header name.

Removes a previously set extra header name. This method can be called only before first write. Trying to call it after the first write will throw an error.

#### `request.write(chunk[, encoding][, callback])`

* `chunk` (String | Buffer) - A chunk of the request body's data. If it is a string, it is converted into a Buffer using the specified encoding.
* `encoding` String (optional) - Used to convert string chunks into Buffer objects. Defaults to 'utf-8'.
* `callback` Function (optional) - Called after the write operation ends.

`callback` is essentially a dummy function introduced in the purpose of keeping similarity with the Node.js API. It is called asynchronously in the next tick after `chunk` content have been delivered to the Chromium networking layer. Contrary to the Node.js implementation, it is not guaranteed that `chunk` content have been flushed on the wire before `callback` is called.

Adds a chunk of data to the request body. The first write operation may cause the request headers to be issued on the wire. After the first write operation, it is not allowed to add or remove a custom header.

#### `request.end([chunk][, encoding][, callback])`

* `chunk` (String | Buffer) (optional)
* `encoding` String (optional)
* `callback` Function (optional)

Sends the last chunk of the request data. Subsequent write or end operations will not be allowed. The `finish` event is emitted just after the end operation.

#### `request.abort()`

Cancels an ongoing HTTP transaction. If the request has already emitted the `close` event, the abort operation will have no effect. Otherwise an ongoing event will emit `abort` and `close` events. Additionally, if there is an ongoing response object,it will emit the `aborted` event.

#### `request.followRedirect()`

Continues any deferred redirection request when the redirection mode is `manual`.