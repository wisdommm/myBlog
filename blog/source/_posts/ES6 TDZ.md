title: ES6中的暂时性死区（TDZ）
---

### 起因

故事要从阮一峰老师的一条微博开始说起。。。

![](http://oatasl78l.bkt.clouddn.com/tdz.jpeg)

这条微博下面有很多的评论，有的厉害的人一眼就指出了这是由于TDZ的原因，并不是什么V8的bug，TDZ是什么？下面我们就来慢慢揭开这层面纱。

<!-- more -->

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

![](http://oatasl78l.bkt.clouddn.com/aaa.png)

所以照现在看来，const和let都是没有变量提升的，对吧。

然后我们来看下面一段代码。

```
let e = 12;
(function(){
console.log(e);
let e = 123;
})()
// 如果说let真的没有变量提升的话，那么这里应该打印出12，OK？实际上如何呢，还是把代码放进浏览器中运行看看吧。
```

![](http://oatasl78l.bkt.clouddn.com/let.png)

怎么会是这样呢，不是说好的没有变量提升吗，怎么会报错啊？下面我们来细细分析。

### let const 详解

在ES6中，最常见的TDZ就是在let/const的使用上，根据ES6中对let/const声明的[章节](http://www.ecma-international.org/ecma-262/6.0/#sec-let-and-const-declarations)，原文是这样说的：

```
The variables are created when their containing Lexical Environment is instantiated but may not be accessed in any way until the variable’s LexicalBinding is evaluated.
```

这段话的意思是说，由let/const声明的变量，当它们包含的词法环境实例化时，才会被创建；且只有在变量的词法绑定已经被赋值后，才能被访问（使用）。

这段话详细地说，就是在程序在其词法环境（也就是其对应的作用域，比如模块，函数，块级作用域等）就行实例化的时候，（let/const声明的）变量就已经被创建了，但是这个时候还不能被访问（此时访问会出现not defined），只有当这个变量的词法绑定被赋值后，才能进行访问这个变量，也就是说当这个变量被赋值后，才能正常访问。所以在变量创建到变量赋值这一段时间内，访问时会报错，这段时间就叫做TDZ（暂时死区）。

因此，从上面看来，似乎let/const声明的变量，还是有变量提升（hoist）的作用的，所以这里也是会让很多人误解的地方，实际上，JS中的变量都有变量提升，这是JS语言中，变量的基本特性，but，因为TDZ的作用，所以let/const的变量提升并没有像var那样明显地得到undefined，而是会直接报错：not defined；而且这很明显是一个运行期间才会出现的问题。

所以上面提到的那个例子就已经说明了let也有变量提升，我们再来分析一下这个例子。

```
let e = 12;
(function(){
// TDZ start
console.log(e);
let e = 123; // TDZ end
})()
```

在这个IIFE函数中，本来console.log的时候，应该现在函数内部找，找不到的话再往外面找；可是由于下面，e又被在这个函数内重新定义赋值了，虽然是在console.log下面，但是由于是有变量提升的，所以在对应的作用域顶端（也就是这个IIFE函数的顶端），是TDZ start，然后直到这个变量被赋值，才是TDZ end，这中间的区域就叫做TDZ；因此，在console的时候由于内部又声明了变量e，所以在内部找到了变量e于是就不用继续往外面找了，因此不会输出12，又因为console的区域刚好处在TDZ的区域内，所以就算变量e被声明了，可是由于TDZ的存在，在变量被赋值之前暂时还不能访问，所以此时访问会出现报错“not defined”。

在ES6中的let与const声明章节的后面几句，说明了有关变量是如何进行初始化的：

```
A variable defined by a LexicalBinding with an Initializer is assigned the value of its Initializer’s AssignmentExpression when the LexicalBinding is evaluated, not when the variable is created. If a LexicalBinding in a let declaration does not have an Initializer the variable is assigned the value undefined when the LexicalBinding is evaluated.
```

这几句是关于变量初始化的过程的。以let/const所声明的变量（const声明的变量叫做固定的变量），必须是经过声明的赋值语句的求值后，才算初始化完成，而不是创建后就初始化完成。let/var声明过后如果没有赋予初始值，那么会赋值为undefined，但是const不行，const声明了变量后一定需要赋初始值的。初始化完成后才代表TDZ的真正结束，这些在作用域中被声明的变量才能被正常的访问。

下面的示例是一个未初始化完成的结果，它还在TDZ中，所以会发生错误，“x is not defined”。

```
let x = x;
```

因为等号右边的x，它在此时还是一个未被初始化完成的变量，实际上我们就在这同一个表达式中要初始化它。

### 函数传参预设值

TDZ作用在ES6中，很明确的就是与区块作用域(block scope)，以及变量/常量的要如何被初始化有关。实际上在许多ES6新特性中都有出现TDZ作用，而另一个常会被提及的是函数的传参预设值中的TDZ作用。

直接上示例把，看下面的代码。

```
function foo(x=y,y=1){
  console.log(y)
}

foo(1) // 这不会有错误

foo(undefined, 1) // 错误 ReferenceError: y is not defined

foo() // 错误 ReferenceError: y is not defined
```

上面这些报错的主要原因是在函数声明时传入的形参初始化的这里，<code>x=y,y=1</code>可以看成<code>let x=y,let y=1</code>，这样应该就会清晰很多。

当然，对于传参预设值的作用域，也是作用域话题里一个常被讨论的话题，讨论这到底是属于“全局作用域”还是“函数中的作用域”，抑或是处于这两者之间的“中介区域”？现在看到比较常见的一种说法就是，它是处于“中介的作用域”，夹在这两者之间，仍然会与其它作用域相互影响。一个比较明显的示例就是，使用其它函数作为作为传参的预设值，这通常会是一个回调函数，一般情况下本没什么，但是涉及到作用域相互影响的时候，就会比较难以理解，比如这个[示例](http://dmitrysoshnikov.com/ecmascript/es6-notes-default-values-of-parameters/#conditional-intermediate-scope-for-parameters)，我将它大致的整理了一下。

```
let x = 1;

function foo(a = 1, b=function(){x = 2;} ){
	let x = 3;
	b();
	console.log(x);

}

foo();

console.log(x);
```

上面这个例子，最后会输出什么呢？

这里有两个console，里面外面各一个，所以会输出两个值，里面的x的值是多少，要看作用域之间相互的影响，而外面的x的值，要看最外围的作用域最后会不会被改变，这也是一个问题。

下面来详细分解一下，函数foo中的x，值可能是2，也可能是3，肯定不是1，这个可以理解吧；而函数外面的那个x，值可能是1，也可能是2，绝不可能是3，这个也可以理解吧。

具体答案是多少？我不知道，或者说没有具体答案，我们可以分别用浏览器和编译器来运行一下代码试试：<strong>chrome下面会分别输出3和2</strong>，<strong>firefox下面会分别输出2和1</strong>，然后我就感到奇怪，再使用Edge来试试，结果发现会输出3与2，最后再使用著名的babel编译器，得出的结果是2与1，暂时看来，结果应该有2个，3与2或者2与1，我们分别来分析一下这两个答案吧，看看是什么问题让浏览器都没有统一的答案。

当结果是3与2的时候，说明b函数中的x=2运行了出来，但是由于中介作用域的影响，所以干扰不到函数中原本的区块作用域，但是直接改变了全局x的值。也就是几本认为了函数预设值中的那个函数中的作用域与全局（或者说是函数外层）有影响。

当结果是2与1的时候，就倒过来，中介作用域影响了函数内层的作用域，而没有影响外层的作用域。

因此，只有当中介作用域有自己独立的作用域的时候，完全与函数区块中的作用域以及函数外层中的作用域毫不相干的时候，那这个时候就会输出3与1，很遗憾，现在都是3与2或者2与1，看来中介作用域还是和函数外层（或者内层相关的），至于具体是外层韩还是内层，这个问题现在还没有一个明确的结果吧，否则各种浏览器的输出也就不会有差异了，不只这一个问题，chrome和firefox的差异还有其它，这里就不细讲了。

不管如何，这个作用域的影响仍然是有争议的，目前并没有统一的答案。这代表ES6虽然标准定好了，但里面的一些新特性仍然有实作细节的差异，未来有可能这些差异才会慢慢一致。但对一般的开发者来说，因为知道了有这些情况，所以要尽量避免，以免产生不必要的麻烦。

### TDZ的其它陷阱

##### typeof语句

```
// 一般来说，typeof 一个未定义的数据，都是undefined，因此typeof也可以作为判断变量是否存在（被赋值）的一种方法，那是因为之前没有let和const。

typeof a; // undefined

typeof b; // b is not defined

let b=1;
```

现在typeof也变得不安全了，你不能随意的对变量使用typeof操作，因为现在会有报错的风险，会变成一个陷阱，由于TDZ的设计，使得变量本就不该在声明前被访问。

#### TDZ期间抛出的错误是运行阶段的错误

TDZ期间的错误，全部是运行阶段才会发生的错误，因为它需要变量／常量初始化的过程，在这个过程中，才会创建出TDZ，且真正运行到那里，调用到函数运行里面的代码时，才会TDZ相关的错误才会被抛出。

#### 不用let/const ？ 

其实没错，只要你声明变量的时候，不管局部还是全部，全部使用var，那么你就根本不用关心TDZ。

可是，在浏览器上的运行效能来看，大多数情况下，[let效能比var强](http://stackoverflow.com/questions/21467642/is-there-a-performance-difference-between-let-and-var)，且会有块级作用域的概念，这会减少因为局部变量／全局变量所引起的问题，而且let对变量有了更高的要求与标准，比如不允许未定义就使用；不允许重复定义等。。。


























































