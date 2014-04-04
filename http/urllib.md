# urllib

![logo](https://raw.github.com/node-modules/urllib/master/logo.png)

Help in opening URLs (mostly HTTP) in a complex world — basic
and digest authentication, redirections, cookies, timeout and more.

If you are using [co](https://github.com/visionmedia/co) or [koa](https://github.com/koajs/koa),
please take a look at [co-urllib](https://github.com/node-modules/co-urllib).

## Install

```bash
$ npm install urllib
```

## Usage

```js
var urllib = require('urllib');

urllib.request('http://cnodejs.org/', { wd: 'nodejs' }, function (err, data, res) {
  if (err) {
    throw err; // you need to handle error
  }
  console.log(res.statusCode);
  console.log(res.headers);
  // data is Buffer instance
  console.log(data.toString());
});
```

## API Doc

### Method: `http.request(url[, options][, callback])`

#### Arguments

- **url** String | Object - The URL to request, either a String or a Object that return by [url.parse](http://nodejs.org/api/url.html#url_url_parse_urlstr_parsequerystring_slashesdenotehost).
- ***options*** Object - Optional
    - ***method*** String - Request method, defaults to `GET`. Could be `GET`, `POST`, `DELETE` or `PUT`. Alias 'type'.
    - ***data*** Object - Data to be sent. Will be stringify automatically.
    - ***content*** String | [Buffer](http://nodejs.org/api/buffer.html) - Manually set the content of payload. If set, `data` will be ignored.
    - ***stream*** [stream.Readable](http://nodejs.org/api/stream.html#stream_class_stream_readable) - Stream to be pipe to the remote. If set, `data` and `content` will be ignored.
    - ***writeStream*** [stream.Writable](http://nodejs.org/api/stream.html#stream_class_stream_writable) - A writable stream to be piped by the response stream. Responding data will be write to this stream and `callback` will be called with `data` set `null` after finished writing.
    - ***dataType*** String - Type of response data. Could be `text` or `json`. If it's `text`, the `callback`ed `data` would be a Buffer. If it's `json`, the `data` of callback would be a parsed JSON Object. Defaults to `text`.
    - ***headers*** Object - Request headers.
    - ***timeout*** Number - Request timeout in milliseconds. Defaults to `exports.TIMEOUT`. Include remote server connecting timeout and response timeout. When timeout happen, will return `ConnectionTimeout` or `ResponseTimeout`.
    - ***auth*** String - `username:password` used in HTTP Basic Authorization.
    - ***digestAuth*** String - `username:password` used in HTTP [Digest Authorization](http://en.wikipedia.org/wiki/Digest_access_authentication).
    - ***agent*** [http.Agent](http://nodejs.org/api/http.html#http_class_http_agent) - HTTP Agent object.
      Set `false` if you does not use agent.
    - ***httpsAgent*** [https.Agent](http://nodejs.org/api/https.html#https_class_https_agent) - HTTPS Agent object.
      Set `false` if you does not use agent.
    - ***ca*** String | Buffer | Array - An array of strings or Buffers of trusted certificates.
      If this is omitted several well known "root" CAs will be used, like VeriSign.
      These are used to authorize connections.
      **Notes**: This is necessary only if the server uses the self-signed certificate
    - ***rejectUnauthorized*** Boolean - If true, the server certificate is verified against the list of supplied CAs.
      An 'error' event is emitted if verification fails. Default: true.
    - ***pfx*** String | Buffer - A string or Buffer containing the private key,
      certificate and CA certs of the server in PFX or PKCS12 format.
    - ***key*** String | Buffer - A string or Buffer containing the private key of the client in PEM format.
      **Notes**: This is necessary only if using the client certificate authentication
    - ***cert*** String | Buffer - A string or Buffer containing the certificate key of the client in PEM format.
      **Notes**: This is necessary only if using the client certificate authentication
    - ***passphrase*** String - A string of passphrase for the private key or pfx.
    - ***followRedirect*** Boolean - follow HTTP 3xx responses as redirects. defaults to false.
    - ***maxRedirects*** Number - The maximum number of redirects to follow, defaults to 10.
    - ***beforeRequest*** Function - Before request hook, you can change every thing here.
    - ***gzip*** Boolean - Accept gzip response content and auto decode it, default is `false`.
- ***callback(err, data, res)*** Function - Optional callback.
    - **err** Error - Would be `null` if no error accured.
    - **data** Buffer | Object - The data responsed. Would be a Buffer if `dataType` is set to `text` or an JSON parsed into Object if it's set to `json`.
    - **res** [http.IncomingMessage](http://nodejs.org/api/http.html#http_http_incomingmessage) - The response.

#### Returns

[http.ClientRequest](http://nodejs.org/api/http.html#http_class_http_clientrequest) - The request.

Calling `.abort()` method of the request stream can cancel the request.

#### Options: `options.data`

When making a request:

```js
urllib.request('http://example.com', {
  method: 'GET',
  data: {
    'a': 'hello',
    'b': 'world'
  }
});
```

For `GET` request, `data` will be stringify to query string, e.g. `http://example.com/?a=hello&b=world`.

For others like `POST`, `PATCH` or `PUT` request,
in defaults, the `data` will be stringify into `application/x-www-form-urlencoded` format
if `Content-Type` header is not set.

If `Content-type` is `application/json`, the `data` will be `JSON.stringify` to JSON data format.

#### Options: `options.content`

`options.content` is useful when you wish to construct the request body by yourself,
for example making a `Content-Type: application/json` request.

Notes that if you want to send a JSON body, you should stringify it yourself:

```js
urllib.request('http://example.com', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  content: JSON.stringify({
    a: 'hello',
    b: 'world'
  })
});
```

It would make a HTTP request like:

```http
POST / HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "a": "hello",
  "b": "world"
}
```

This exmaple can use `options.data` with `application/json` content type:

```js
urllib.request('http://example.com', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  data: {
    a: 'hello',
    b: 'world'
  }
});
```

#### Options: `options.stream`

Uploads a file with [formstream](https://github.com/fengmk2/formstream):

```js
var urllib = require('urllib');
var formstream = require('formstream');

var form = formstream();
form.file('file', __filename);
form.field('hello', '你好urllib');

var req = urllib.request('http://my.server.com/upload', {
  method: 'POST',
  headers: form.headers(),
  stream: form
}, function (err, data, res) {
  // upload finished
});
```

### Response: `res.aborted`

If the underlaying connection was terminated before `response.end()` was called,
`res.aborted` should be `true`,
and return `RemoteSocketClosedError` error.

```js
var server = require('http').createServer(function (req, res) {
  req.resume();
  req.on('end', function () {
    res.write('foo haha\n');
    setTimeout(function () {
      res.write('foo haha 2');
      setTimeout(function () {
        res.socket.end();
      }, 300);
    }, 200);
    return;
  });
});

urllib.request('http://127.0.0.1:1984/socket.end', function (err, data, res) {
  err.name.should.equal('RemoteSocketClosedError');
  err.message.should.equal('Remote socket was terminated before `response.end()` was called');
  data.toString().should.equal('foo haha\nfoo haha 2');
  should.ok(res.aborted);
  done();
});
```
