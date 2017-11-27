---
title: javascript异步编程（续）
---

之前那篇文章讲解了javascript异步式编程的几个方法，比如有回调函数，promise，发布／订阅模式，generator函数等，每一种方式，在其广泛使用的时期，都是很完善的一种方式，不过由于时代的进步，技术的发展，缺点总会暴露出来，更好更完善的方法也会被创造出来。

这篇文章，我就想来说说<code>async</code>函数。

ES7意见征集稿中提供了<code>async</code>函数，这是什么？其实可以用一句话来概括，<code>async</code>函数就是Generator函数的语法糖。

我们从Generator函数开始，代码如下。

<!-- more -->

```
// 这是采用fs模块来异步读取文件
var fs = require('fs');

var readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) reject(error);
      resolve(data);
    });
  });
};

// 这是使用generator函数的方法，依次读取两个文件
var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

// 下面是async函数的写法
var asyncReadFile = async function (){
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

其实一比较，差异很明显，<code>async</code>函数就是将generator函数的星号替换成了<code>async</code>，将<code>yield</code>替换成了<code>await</code>。

仅此而已！仅此而已？

虽然加入了比较好看的语法糖，不过一个新的方法如果只是比老的方法命名好听，有什么意义呢？下面我来说一下<code>async</code>函数和generator函数的主要区别吧。

1. 更好的语义。既然都看出来了，那最明显的当然是标志符换了，也就是更加语义化了，<code>await</code>和<code>async</code>比星号跟yield更加让人好理解。

2. 更好的适用性。generator函数的执行必须依靠执行器，或者人为的多次调用next方法才能继续执行下去，而<code>async</code>是自带执行器的，这是什么意思呢？意思就是你再也不用管什么next了，也不用管怎么异步执行了，你完完全全就可以把<code>async</code>函数就当做普通的函数来看来，当你想执行的时候，只需要一句<code>var result = async();</code>就可以了，她会自动调用<code>async</code>函数并且自动执行，输出最后结果，你现在可以将什么next方法啊，co模块这些全部放一边了。

3. 返回值是promise。<code>async</code>函数的返回值是promise对象，这比generator函数的返回一个Iterator对象方便多了，你可以使用promise里的then方法来进行下一步的操作。而<code>async</code>函数完全可以看作是多个异步的操作，包装成了一个promise对象，而await就是then方法的语法糖。

下面是一些<code>async</code>函数的具体使用示例。

1，下面代码中，函数f中return的数据，会被promise的then方法的回调函数获取到，所以then方法回调函数里的参数v就是指代的f函数的return的数据，所以控制台才会打印出hello world

```
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// 打开控制台，你就会看见有一句hello world（前提是你的浏览器支持async）
```

2，<code>async</code>函数内部抛出错误，会导致返回的promise对象变为reject状态，抛出的错误对象，会被catch方法的回调函数捕捉到。

```
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
```

3，<code>async</code>函数返回的promise对象，必须要等到内部await命令的promise对象执行完，才会发生状态的改变，换句话说，也就是<code>async</code>函数内部的await命令执行完了，才会执行then方法的回调函数。如下面的例子。

```
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

4，在正常情况下，await后面应该是一个promise对象，如果不是，那么会被转变为一个立即resolve的promise对象。

```
async function f() {
  return await 123;
}

f().then(v => console.log(v))
// 123，这里await后面的123虽然不是promise对象，不过它会转变为promise对象并立即被resolve，所以在后面的then方法的回调函数里，依然可以正常的运行。
```

5，<code>async</code>函数返回的promise对象，如果变为了reject状态，则reject的参数会被catch方法的回调函数接收到，示例如下：

```
async function f() {
  await Promise.reject('出错了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出错了。上面的await前面没有return语句，但是reject状态的参数依然传入了catch方法的回调函数里，所以说这里有没有return都是不影响的，不过为了统一与规范，还是建议加上。
```

6，只有程序发现了一个promise函数的状态变为了reject，那么后面的代码将不会执行。

```
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 这句将不会执行
}

// 所以为了避免这种情况发生，可以将第一个await放在try...catch...语句中，这样代码的执行就不会被中断。

async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world

// 另一种解决办法就是在await后面的promise对象加上一个catch方法，来处理前面可能会出现的错误，代码如下。

async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出错了
// hello world

// 当然，我们也可以将多个await命令放进catch方法里，这段代码就省略了。。。
```

7，如果await后面的异步操作出错，那么就等同于promise对象状态变为reject。

```
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出错了
```

上面说了那么多的示例，都是比较片面的讲解使用中的注意事项，完整的示例来讲解<code>async</code>函数的使用方法，所以我下面会主要讲<code>async</code>函数的使用。

1，上面代码是一个获取股票报价的函数，函数前面的async关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个Promise对象。

```
async function getStockPriceByName(name) {
  var symbol = await getStockSymbol(name);
  var stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
// 最后打印出的这个result就是getStockPriceByName函数中return的stockPrice。
```

2，下面代码指定50毫秒以后，输出"hello world"。

```
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value)
}

asyncPrint('hello world', 50);
```

十一国庆节来公司基本没人，后背也感到一阵阵凉风，这篇博客就先讲到这里吧，有什么完善的地方后面我会再继续补充的。






















































































