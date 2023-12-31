---
category: 专业知识
tags:
  - Node.js
date: 2020-06-25
title: nodejs流
vssue-title: nodejs流
---

## 引子

> 在编写代码时，我们应该有一些方法将程序像连接水管一样连接起来 -- 当我们需要获取一些数据时，可以去通过"拧"其他的部分来达到目的。这也应该是 IO 应有的方式。 -- [Doug McIlroy. October 11, 1964](https://www.bell-labs.com/usr/dmr/www/mdmpipe.html)

从[早先的 unix](http://www.youtube.com/watch?v=tc4ROCJYbm0)开始，stream 便开始进入了人们的视野，在过去的几十年的时间里，它被证明是一种可依赖的编程方式，它可以将一个大型的系统拆成一些很小的部分，并且让这些部分之间完美地进行合作。在 unix 中，我们可以使用`|`符号来实现流。在 node 中，node 内置的[stream 模块](http://nodejs.org/docs/latest/api/stream.html)已经被多个核心模块使用，同时也可以被用户自定义的模块使用。和 unix 类似，node 中的流模块的基本操作符叫做`.pipe()`，同时你也可以使用一个后压机制来应对那些对数据消耗较慢的对象。

在 node 中，流可以帮助我们将事情的重点分为几份，因为使用流可以帮助我们将实现接口的部分分割成一些连续的接口，这些接口都是可重用的。接着，你可以将一个流的输出口接到另一个流的输入口，然后使用使用一些库来对流实现高级别的控制。

对于小型程序设计(small-program design)以及 unix 哲学来说，流都是一个重要的组成部分，但是除此之外还有一些重要的事情值得我们思考。永远要记得：双鸟在林不如一鸟在手。

## 为什么应该使用流

在 node 中，I/O 都是异步的，所以在和硬盘以及网络的交互过程中会涉及到传递回调函数的过程。你之前可能会写出这样的代码：

```js
var http = require("http");
var fs = require("fs");

var server = http.createServer(function(req, res) {
  fs.readFile(__dirname + "/data.txt", function(err, data) {
    res.end(data);
  });
});
server.listen(8000);
```

上面的这段代码并没有什么问题，但是在每次请求时，我们都会把整个`data.txt`文件读入到内存中，然后再把结果返回给客户端。想想看，如果`data.txt`文件非常大，在响应大量用户的并发请求时，程序可能会消耗大量的内存，这样很可能会造成用户连接缓慢的问题。

其次，上面的代码可能会造成很不好的用户体验，因为用户在接收到任何的内容之前首先需要等待程序将文件内容完全读入到内存中。

所幸的是，`(req,res)`参数都是流对象，这意味着我们可以使用一种更好的方法来实现上面的需求：

```js
var http = require("http");
var fs = require("fs");

var server = http.createServer(function(req, res) {
  var stream = fs.createReadStream(__dirname + "/data.txt");
  stream.pipe(res);
});
server.listen(8000);
```

在这里，`.pipe()`方法会自动帮助我们监听`data`和`end`事件。上面的这段代码不仅简洁，而且`data.txt`文件中每一小段数据都将源源不断的发送到客户端。

除此之外，使用`.pipe()`方法还有别的好处，比如说它可以自动控制后端压力，以便在客户端连接缓慢的时候 node 可以将尽可能少的缓存放到内存中。

想要将数据进行压缩？我们可以使用相应的流模块完成这项工作!

```js
var http = require("http");
var fs = require("fs");
var oppressor = require("oppressor");

var server = http.createServer(function(req, res) {
  var stream = fs.createReadStream(__dirname + "/data.txt");
  stream.pipe(oppressor(req)).pipe(res);
});
server.listen(8000);
```

通过上面的代码，我们成功的将发送到浏览器端的数据进行了 gzip 压缩。我们只是使用了一个 oppressor 模块来处理这件事情。

一旦你学会使用流 api，你可以将这些流模块像搭乐高积木或者像连接水管一样拼凑起来，从此以后你可能再也不会去使用那些没有流 API 的模块获取和推送数据了。

## 流模块基础

在 node 中，一共有五种类型的流：readable,writable,transform,duplex 以及"classic"

### pipe

无论哪一种流，都会使用`.pipe()`方法来实现输入和输出。

`.pipe()`函数很简单，它仅仅是接受一个源头`src`并将数据输出到一个可写的流`dst`中：

```js
src.pipe(dst);
```

`.pipe(dst)`将会返回`dst`因此你可以链式调用多个流:

```js
a.pipe(b)
  .pipe(c)
  .pipe(d);
```

上面的代码也可以等价为：

```js
a.pipe(b);
b.pipe(c);
c.pipe(d);
```

这和你在 unix 中编写流代码很类似：

```
a | b | c | d
```

只不过此时你是在 node 中编写而不是在 shell 中！

### readable 流

Readable 流可以产出数据，你可以将这些数据传送到一个 writable，transform 或者 duplex 流中，只需要调用`pipe()`方法:

```js
readableStream.pipe(dst);
```

#### 创建一个 readable 流

现在我们就来创建一个 readable 流！

```js
var Readable = require("stream").Readable;

var rs = new Readable();
rs.push("beep ");
rs.push("boop\n");
rs.push(null);

rs.pipe(process.stdout);
```

下面运行代码：

```
$ node read0.js
beep boop
```

在上面的代码中`rs.push(null)`的作用是告诉`rs`输出数据应该结束了。

需要注意的一点是我们在将数据输出到`process.stdout`之前已经将内容推送进 readable 流`rs`中，但是所有的数据依然是可写的。

这是因为在你使用`.push()`将数据推进一个 readable 流中时，一直要到另一个东西来消耗数据之前，数据都会存在一个缓存中。

然而，在更多的情况下，我们想要的是当需要数据时数据才会产生，以此来避免大量的缓存数据。

我们可以通过定义一个`._read`函数来实现按需推送数据:

```js
var Readable = require("stream").Readable;
var rs = Readable();

var c = 97;
rs._read = function() {
  rs.push(String.fromCharCode(c++));
  if (c > "z".charCodeAt(0)) rs.push(null);
};

rs.pipe(process.stdout);
```

代码的运行结果如下所示:

```
$ node read1.js
abcdefghijklmnopqrstuvwxyz
```

在这里我们将字母`a`到`z`推进了 rs 中，但是只有当数据消耗者出现时，数据才会真正实现推送。

`_read`函数也可以获取一个`size`参数来指明消耗者想要读取多少比特的数据，但是这个参数是可选的。

需要注意到的是你可以使用`util.inherit()`来继承一个 Readable 流。

为了说明只有在数据消耗者出现时，`_read`函数才会被调用，我们可以将上面的代码简单的修改一下：

```js
var Readable = require("stream").Readable;
var rs = Readable();

var c = 97 - 1;

rs._read = function() {
  if (c >= "z".charCodeAt(0)) return rs.push(null);

  setTimeout(function() {
    rs.push(String.fromCharCode(++c));
  }, 100);
};

rs.pipe(process.stdout);

process.on("exit", function() {
  console.error("\n_read() called " + (c - 97) + " times");
});
process.stdout.on("error", process.exit);
```

运行上面的代码我们可以发现如果我们只请求 5 比特的数据，那么`_read`只会运行 5 次：

```
$ node read2.js | head -c5
abcde
_read() called 5 times
```

在上面的代码中，`setTimeout`很重要，因为操作系统需要花费一些时间来发送程序结束信号。

另外,`process.stdout.on('error',fn)`处理器也很重要，因为当`head`不再关心我们的程序输出时，操作系统将会向我们的进程发送一个`SIGPIPE`信号，此时`process.stdout`将会捕获到一个`EPIPE`错误。

上面这些复杂的部分在和操作系统相关的交互中是必要的，但是如果你直接和 node 中的流交互的话，则可有可无。

如果你创建了一个 readable 流，并且想要将任何的值推送到其中的话，确保你在创建流的时候指定了 objectMode 参数,`Readable({ objectMode: true })`。

#### 消耗一个 readable 流

大部分时候，将一个 readable 流直接 pipe 到另一种类型的流或者使用 through 或者 concat-stream 创建的流中，是一件很容易的事情。但是有时我们也会需要直接来消耗一个 readable 流。

```js
process.stdin.on("readable", function() {
  var buf = process.stdin.read();
  console.dir(buf);
});
```

代码运行结果如下所示：

```bash
$ (echo abc; sleep 1; echo def; sleep 1; echo ghi) | node consume0.js
<Buffer 61 62 63 0a>
<Buffer 64 65 66 0a>
<Buffer 67 68 69 0a>
null
```

当数据可用时，`readable`事件将会被触发，此时你可以调用`.read()`方法来从缓存中获取这些数据。

当流结束时，`.read()`将返回`null`，因为此时已经没有更多的字节可以供我们获取了。

你也可以告诉`.read()`方法来返回`n`个字节的数据。虽然所有核心对象中的流都支持这种方式，但是对于对象流来说这种方法并不可用。

下面是一个例子，在这里我们制定每次读取 3 个字节的数据：

```js
process.stdin.on("readable", function() {
  var buf = process.stdin.read(3);
  console.dir(buf);
});
```

运行上面的例子，我们将获取到不完整的数据:

```bash
$ (echo abc; sleep 1; echo def; sleep 1; echo ghi) | node consume1.js
<Buffer 61 62 63>
<Buffer 0a 64 65>
<Buffer 66 0a 67>
```

这是因为多余的数据都留在了内部的缓存中，因此这个时候我们需要告诉 node 我们还对剩下的数据感兴趣，我们可以使用`.read(0)`来完成这件事：

```js
process.stdin.on("readable", function() {
  var buf = process.stdin.read(3);
  console.dir(buf);
  process.stdin.read(0);
});
```

到现在为止我们的代码和我们所期望的一样了！

```bash
$ (echo abc; sleep 1; echo def; sleep 1; echo ghi) | node consume2.js
<Buffer 61 62 63>
<Buffer 0a 64 65>
<Buffer 66 0a 67>
<Buffer 68 69 0a>
```

我们也可以使用`.unshift()`方法来放置多余的数据。

使用`unshift()`方法能够防止我们进行不必要的缓存拷贝。在下面的代码中我们将创建一个分割新行的可读解析器:

```js
var offset = 0;

process.stdin.on("readable", function() {
  var buf = process.stdin.read();
  if (!buf) return;
  for (; offset < buf.length; offset++) {
    if (buf[offset] === 0x0a) {
      console.dir(buf.slice(0, offset).toString());
      buf = buf.slice(offset + 1);
      offset = 0;
      process.stdin.unshift(buf);
      return;
    }
  }
  process.stdin.unshift(buf);
});
```

代码的运行结果如下所示：

```bash
$ tail -n +50000 /usr/share/dict/american-english | head -n10 | node lines.js
'hearties'
'heartiest'
'heartily'
'heartiness'
'heartiness\'s'
'heartland'
'heartland\'s'
'heartlands'
'heartless'
'heartlessly'
```

当然，已经有很多这样的模块比如 split 来帮助你完成这件事情，你完全不需要自己写一个。

### writable 流

一个 writable 流指的是只能流进不能流出的流:

```js
src.pipe(writableStream);
```

#### 创建一个 writable 流

只需要定义一个`._write(chunk,enc,next)`函数，你就可以将一个 readable 流的数据释放到其中：

```js
var Writable = require("stream").Writable;
var ws = Writable();
ws._write = function(chunk, enc, next) {
  console.dir(chunk);
  next();
};

process.stdin.pipe(ws);
```

代码运行结果如下所示：

```
$ (echo beep; sleep 1; echo boop) | node write0.js
<Buffer 62 65 65 70 0a>
<Buffer 62 6f 6f 70 0a>
```

第一个参数，`chunk`代表写进来的数据。

第二个参数`enc`代表编码的字符串，但是只有在`opts.decodeString`为`false`的时候你才可以写一个字符串。

第三个参数，`next(err)`是一个回调函数，使用这个回调函数你可以告诉数据消耗者可以写更多的数据。你可以有选择性的传递一个错误对象`error`，这时会在流实体上触发一个`emit`事件。

在从一个 readable 流向一个 writable 流传数据的过程中，数据会自动被转换为`Buffer`对象，除非你在创建 writable 流的时候制定了`decodeStrings`参数为`false`,`Writable({decodeStrings: false})`。

如果你需要传递对象，需要指定`objectMode`参数为`true`，`Writable({ objectMode: true })`。

#### 向一个 writable 流中写东西

如果你需要向一个 writable 流中写东西，只需要调用`.write(data)`即可。

process.stdout.write('beep boop\n');

为了告诉一个 writable 流你已经写完毕了，只需要调用`.end()`方法。你也可以使用`.end(data)`在结束前再写一些数据。

```js
var fs = require("fs");
var ws = fs.createWriteStream("message.txt");

ws.write("beep ");

setTimeout(function() {
  ws.end("boop\n");
}, 1000);
```

运行结果如下所示:

```
$ node writing1.js
   $ cat message.txt
beep boop
```

如果你需要调整内部缓冲区大小，那么需要在创建可写流对象时设置`highWaterMark`。在调用`.write()`方法返回 false 时，说明写入的数据大小超过了该值。

为了避免读写速率不匹配而造成内存上涨，可以监听`drain`事件，等待可写流内部缓存被清空再继续写入。

### transform 流  

你可以将 transform 流想象成一个流的中间部分，它可以读也可写，但是并不保存数据，它只负责处理流经它的数据。

### duplex 流

Duplex 流是一个可读也可写的流，就好像一个电话，可以接收也可以发送语音。一个 rpc 交换是一个 duplex 流的最好的例子。如果你看到过下面这样的代码：

```js
a.pipe(b).pipe(a);
```

那么你需要处理的就是一个 duplex 流对象。

### classic 流

Classic 流是一个古老的接口，最早出现在 node 0.4 中。虽然现在不怎么用，但是我们最好还是来了解一下它的工作原理。

无论何时，只要一个流对象注册了一个`data`监听器，它就会自动的切换到`classic`模式，并且根据旧 API 的方式运行。

#### classic readable 流

Classic readable 流只是一个事件发射器，当有数据消耗者出现时发射`emit`事件，当输出数据完毕时发射`end`事件。

我们可以同构检查`stream.readable`来检查一个 classic 流对象是否可读。

下面是一个简单的 readable 流对象的例子，程序的运行结果将会输出`A`到`J`：

```js
var Stream = require("stream");
var stream = new Stream();
stream.readable = true;

var c = 64;
var iv = setInterval(function() {
  if (++c >= 75) {
    clearInterval(iv);
    stream.emit("end");
  } else stream.emit("data", String.fromCharCode(c));
}, 100);

stream.pipe(process.stdout);
```

运行结果如下所示:

```
$ node classic0.js
ABCDEFGHIJ
```

为了从一个 classic readable 流中读取数据，你可以注册`data`和`end`监听器。下面是一个使用旧 readable 流方式从`process.stdin`中读取数据的例子:

```js
process.stdin.on("data", function(buf) {
  console.log(buf);
});
process.stdin.on("end", function() {
  console.log("**END**");
});
```

运行结果如下所示:

```
$ (echo beep; sleep 1; echo boop) | node classic1.js
<Buffer 62 65 65 70 0a>
<Buffer 62 6f 6f 70 0a>
**END**
```

需要注意的一点是当你在一个流对象上注册了一个`data`监听器，你就将这个流放在了兼容模式下，此时你不能使用两个 stream2 的 api。

如果你自己创建流对象，永远不要绑定`data`和`end`监听器。如果你需要和旧版本的流兼容，最好使用第三方库来实现`.pipe()`方法。

例如，你可以使用 through 模块来避免显式的使用`data`和`end`监听器:

```js
var through = require("through");
process.stdin.pipe(through(write, end));

function write(buf) {
  console.log(buf);
}
function end() {
  console.log("**END**");
}
```

程序运行结果如下所示:

```
$ (echo beep; sleep 1; echo boop) | node through.js
<Buffer 62 65 65 70 0a>
<Buffer 62 6f 6f 70 0a>
**END**
```

你也可以使用 concat-stream 模块来将整个流的内容缓存起来:

```js
var concat = require("concat-stream");
process.stdin.pipe(
  concat(function(body) {
    console.log(JSON.parse(body));
  })
);
```

程序运行结果如下所示:

```
$ echo '{"beep":"boop"}' | node concat.js
{ beep: 'boop' }
```

Classic readable 流拥有`.pause()`和`.resume()`逻辑来暂停一个流，但是这都是可选的。如果你想要使用`.pause()`和`.resume()`方法，你应该使用 through 模块来帮助你处理缓存。

#### classic writable 流

Classic writable 流非常简单。其中只定义了`.write(buf)`，`.end(buf)`，以及`.desctory()`方法。其中`.end(buf)`的参数 buf 是可选参数，但是一般来说 node 程序员还是喜欢使用`.end(buf)`这种写法。

## 接下来读什么

- [node 核心 stream 模块文档](http://nodejs.org/docs/latest/api/stream.html#stream_stream)
- 你可以使用[readable-stream](https://npmjs.org/package/readable-stream)模块来确保你的 stream2 代码兼容 node 0.8 及其之前的代码。在你`npm install readable-stream`之后直接`require('readable-stream')`而不要`require('stream')`。

---

本文参考自 stream-handbook，原文地址https://github.com/substack/stream-handbook