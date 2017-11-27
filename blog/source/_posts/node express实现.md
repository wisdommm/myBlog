---
title: express实现
---

express是node里面一个十分常用的库，用它来搭建服务器，事半功倍，且小巧灵活，有足够多的特性，用来进行各种网页开发。

我们先来看下express的使用的，毕竟知己知彼，百战不殆。

```
//引入express，确保你的项目依赖了express库，否则的话先npm install一下
var express = require('express');
//执行express函数，
var app = express();
// 使用get方法，当请求的路由是hello时，返回hello
app.get('/hello', function (req, res) {
  res.send('hello');
});
// 当请求的路由是world时，返回world
app.get('/world', function (req, res) {
  res.send('world');
});
// 当请求的路由是其它时，返回“没有找到匹配的路径”
app.get('*', function (req, res) {
	res.setHeader('content-type','text/plain;charset=utf8');
	res.end('没有找到匹配的路径');
});
// 最后监听端口
var server = app.listen(3000, function () {
  console.log('正在监听3000端口');
});
```

上面或许就是express最常见，也是最基本的使用吧，启动了一个服务器并且监听3000端口，对/hello的路由返回“hello”，对/world的路由返回“world”，对其他的路由返回“没有找到匹配的路径”。

示例看完了，下面我们来看看具体的实现吧，在这之前，我想说一下http这个模块，毕竟express就是对http进行了一下封装，既然想看看express的内部，当然就离不开http了。

<!-- more -->

```
var http = require('http');
http.createServer(function (request, response) {
  response.writeHead(200, {'Content-Type': 'text/plain'});
  response.end('Hello World\n');
}).listen(3000, '127.0.0.1');
console.log('正在监听3000端口。');
```

就上面这几行，就已经成功地实现了一个简单的服务器了，我们可以分拆一下。
```
http.createServer(function(...){...}).listen(3000);
```

现在，我们再回过头来看看express。

```
// 首先，声明express函数
var express = function () {};
// 它的内部应该有一个app函数，用来作为实例。
var express = function () {
	var app = function(...){...}
};
// 这个实例还应该有很多的方法，比如get，listen等。（本来我是想用es5的方式写的，后来发现好久没用那种方法，写function写得太累了。。。）
var express = ()=> {
	var app = (req,res)=>{...};
	app.listen = (...)=>{...};
	app.get = (...)=>{...};
};
// 最后，为了能够完成链式调用，所以我们返回这个实例app，然后exports一下，就完成了。
var express = ()=> {
	var app = (...)=>{...};
	app.listen = (...)=>{...};
	app.get = (...)=>{...};
	return app;
};
module.exports = exports = express;
```

这就是express基本的框架，如果后续有什么扩展的，也可以直接在里面增加一个函数就好了，``` app.xxx = (...)=>{...} ```，接下来，我们来慢慢分析里面的构造。

先看一个最简单的吧，listen方法。
```
app.listen = function (port) {
	require('http').createServer(app).listen(port);
};
```

是不是发现了什么？再上去看一眼http，是不是很相似啊。。。没错，就是这样的。

```
http.createServer(function(...){...}).listen(3000);
```

而中间的app函数，对应的就是http.createServer的参数了。
感觉这个listen函数应该不需要多讲。
























































