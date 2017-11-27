------
title: promise 详解
------

虽然之前写了javascript异步编程，不过我发现周围的部分小伙伴，当然也包括我，对promise的认识还是只停留在基础是使用，类似于ajax这种的使用，或者会用<code>promise.then()</code>，对于原生的Promise对象只了解一点甚至不了解，我也是模模糊糊地了解，所以准备写一篇关于<code>promise</code>的博客，顺便也借此机会一举击破promise，不想再搞得模模糊糊的了，彻底得解决掉这个东西。

在去年，准确的说是在2015年6月（我专门查了的，肯定没有错），ECMAScript6的正式版发布了。ECMAScript 是 JavaScript 语言的国际标准，JavaScript 是 ECMAScript 的实现。ES6 的目标，是使得 JavaScript 语言可以用来编写大型的复杂的应用程序，成为企业级开发语言。

ES6的改变还是算不小的，其中就包含了提供Promise对象。

Promise是什么？首先，它是一个对象，它的作用是用来进行异步操作，代表着某个异步进行的事件的结果。

Promise对象有以下两个特点：

（1），对象的状态不受外界的影线。Promise对象代表一个异步的操作，它包含了三种状态，分别是：Pending（进行中），Resolve（已完成），Reject（已失败）。而异步操作的结果，决定了它将会输出哪一种状态，任何其它操作都无法改变这个状态。这也是Promise名字的由来，也就是承诺的意思，代表一定会去做。

（2），一旦状态改变，就不会再变。虽然Promise不受外界影响状态，不过状态是会在内部进行改变的，它有两种改变状态的可能，一种是从进行中变为已完成（Pending to Resolve），另一种是从进行中变为已失败（Pending to Reject）。除此之外，状态不会再发生其它的变化。

有了 Promise 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise 对象提供统一的接口，使得控制异步操作更加容易。

#### CODE

"talk is cheap,show me the code".

这是程序员界比较常见的一句话，上面很多概念啊这些死知识讲解完了，下面才开始进入正题，开始代码演示。

由于Promise已经是JS原生对象了，所以直接打开浏览器的控制台，输入Promise，就可以看到一个function Promise，如果报错的话，说明你的浏览器该升级了。

<!-- more -->

```javascript
var promise = new Promise(function(resolve, reject) {
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

如上面的基本代码所示，Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject方法，如果异步操作成功了，则调用resolve方法，将Promise对象的状态从Pending改为Resolve，如果异步操作失败，则调用reject方法，将Promise对象的状态从Pending改为Reject。这是promise最基本的使用方法。

promise的神奇之处在于给了我们return和throw，每个promise都会提供一个then函数和一个catch函数，实际上是then(null,...)函数。基本代码如下：

```
promiseOne.then(function(value){
  alert(value); // 这个value是promiseOne中resolve传递出来的值。
  // do something
});
```

在这个then函数里面，我们可以做三件事，1，return 另一个promise；2，return 一个同步的值；3，throw 一个同步异常，‘ throw new Error('') ’。

下面是一些Promise的基本API。

```
1,Promise.resolve()
2,Promise.reject()
3,Promise.prototype.then()
4,Promise.prototype.catch()
5,Promise.all([functionOne,functionTwo,...]) // 所有的完成
6,Promise.race([functionOne,functionTwo,...]) // 竞速，完成一个即可
```

<code>Promise.resolve</code>和<code>Promise.reject</code>是Promise的静态方法，使用方法如下：

```
Promise.resolve("value").then(function(v) {
  console.log(v,1); 
}, function(v) {
  // 不会被调用
console.log(v,2);
});
// 这时控制台会输出 value,1

Promise.resolve("value").then(function(v) {
  console.log(v,1); 
}, function(v) {
  console.log(v,2);
});
// 这时控制台会输出 value,2

```

也就是说，Promise.resolve里传入的参数或者promise对象，会被then方法接收，Promise.reject里传入的参数或者promise对象，会被catch方法接收，这一点在上面的<code>new Promise((resolve,reject)=>{if(...){resolve(v)}else{reject(v)}})</code>已经讲过了，意思是一样的，不同之处在于使用的方式不同，下面的方法是直接调用的promise里的静态方法，而没有用到promise这个构造函数。

除了返回基本数据外，一个promise里面也可以返回另一个promise对象。

```
var original = Promise.resolve(true);
var cast = Promise.resolve(original);
cast.then(function(v) {
  console.log(v); 
});
// 控制台会输出 true
```

我们也可以在promise中抛出异常，这里有同步代码异常和异步代码异常两种。

这是同步代码异常，直接调用promise的reject方法，传入一个error就好了：

```
Promise.reject(new Error("什么鬼"));
```

下面是异步代码异常：

```
new Promise(function (resolve, reject) {
    throw new Error('悲剧了，又出 bug 了');
  }).catch(function(err){
    console.log(err);
  });
```

如果有多个callback函数连续回调，可以分别写成promise的状态，然后在每一个promise对象中return下一个promise，for example:

```
var p1 = new Promise((resolve,reject)=>{
  resolve(1);
});

var p2 = new Promise((resolve,reject)=>{
  resolve(p1);
});

var p3 = new Promise((resolve,reject)=>{
  resolve(p2);
});

p3.then(function(v) {
  console.log(v); // true
});

// 上面是promise链式调用的一种情况，下面是另一种情况，其核心原因都是差不多的。

var p1 = (resolve,reject)=>{
  resolve(1);
}

var p2 = (resolve,reject)=>{
  resolve(2);
}

var p3 = (resolve,reject)=>{
  resolve(3);
}

new Promise(p3).then(function(v) {
  console.log(v); 
  return new Promise(p2).then((v)=>{
    console.log(v)
    return new Promise(p1).then((v)=>{
      console.log(v)
    })
  })
})
```