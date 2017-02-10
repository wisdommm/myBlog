title: ES6中的暂时性死区（TDZ）
---

### 起因

故事要从阮一峰老师的一条微博开始说起。。。

![](http://oatasl78l.bkt.clouddn.com/tdz.jpeg)

这条微博下面有很多的评论，有的厉害的人一眼就指出了这是由于TDZ的原因，并不是什么V8的bug，TDZ是什么？下面我们就来慢慢揭开这层面纱。

### 变量提升

先从最简单的变量提升说起。

```
console.log(a);
var a = 123;

// 这里会输出undefined，原因是上面这段代码和下面的这段代码是等价的

var a;
console.log(a);
a = 123;
```

在ES6中，除了var关键字意外，还多了let和const关键字，有什么差异呢？

先说const吧，const的意思是定义静态常量，是不可变的，所以我们可以先来开脑洞想想，当const遇上变量提升，会发生什么？

```
console.log(a);
const a = 123;

// 如果const的变量提升存在，则上面代码应该和下面代码等价

const a;
console.log(a);
a = 123;
```

想必大家都发现问题了吧，很明显，前面说了const是定义的静态常量，是不可变的，假设上面的代码是对的，那么最开始a应该是undefined，可是后面却被再次赋值成了123，和const定义的静态常量不可改变相违背，所以这里就有问题，所以const没有变量提升？至少现在看起来，const是不应该有变量提升的。

然后再来看看let关键字，和上面一样，当let遇见变量提升，又会怎样呢？老规矩，上代码吧。

```
console.log(a);
let a = 123;

// 如果let的变量提升存在，则上面代码应该和下面代码等价

let a;
console.log(a);
a = 123;
```

这段代码，至少看起来是没有问题的，不是吗？可是实际有问题没有，还是要放进浏览器中运行一下才知道，所以我们把这段代码放进chrome里看看。

可惜，似乎很不幸。。。

![](http://oatasl78l.bkt.clouddn.com/2728034159022278457.jpg)

所以照现在看来，const和let都是没有变量提升的，对吧。

然后我们来看下面一段代码。

```
let e = 12;
function foo(){
console.log(e);
let e = 123;
}
foo();
// 如果说let真的没有变量提升的话，那么这里应该打印出12，OK？实际上如何呢，还是把代码放进浏览器中运行看看吧。
```

![](http://oatasl78l.bkt.clouddn.com/let.png)

怎么会是这样呢？



























