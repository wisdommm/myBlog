---
title: react, es5 or es6 ?
---

随着es6的逐渐普及，新的语法糖，对我们以前写js的习惯也有着不小的冲击，比如说我好久都没写过function这个词语了，取而代之的是“=>”箭头函数，var也基本不用了，现在用let，以及其他等等。

但是对于写react，es5和es6的变化，还是稍微有一些不同的，“talk is cheap,show the code”。

这是es5的写法：

<!-- more -->

```
var InputControlES5 = React.createClass({
  defaultProps：{
    initialValue：''
  },
  propTypes：{
    initialValue：React.PropTypes.string
  }, 
  // Set up initial state
  getInitialState：function() {
    return {
      text：this.props.initialValue || 'placeholder' 
    };
  },
  handleChange：function(event) {
    this.setState({
      text：event.target.value
    });
  },
  render：function() {
    return (
      <div>
        Type something：        
        <input onChange={this.handleChange} 
          value={this.state.text} />
        </div> 
      );
    }
});
```

下面是es6的写法：

```
class InputControlES6 extends React.Component { 
  constructor(props) {
    super(props); 
    // Set up initial state
    this.state = {
      text：props.initialValue || 'placeholder' 
    };
    // Functions must be bound manually with ES6 classes
    this.handleChange = this.handleChange.bind(this); 
  } 
  handleChange(event) {
    this.setState({ 
      text：event.target.value
    });
  } 
  render() {
    return (
      <div>
        Type something：
        <input onChange={this.handleChange} value={this.state.text} />
      </div>
    );
  }
}

InputControlES6.propTypes = { 
  initialValue：React.PropTypes.string
};

InputControlES6.defaultProps = {
  initialValue：''
};
```

一下是一些关键的不同点：

#### 创建方式不同

es5是通过creatClass来创建的react组件，而es6是通过class extends来创建的，或许这是最直接的嫩个发现的差异吧，毕竟这在第一行。

#### 函数的绑定

话说以前，我修改过别人用es6写的react代码，我习惯在每个函数后面加上一个逗号，然后我在render函数中增加了一个事件，对应执行的函数是this.xxx，结果呢？报了一堆的错（包括之前的逗号），“this.xxx is undefined.”，当时我还不知道es6与es5的差异，当时就一脸无辜的表情，然后没办法，我把别人的代码全部用es5重写了一遍，并且当时表示以后都会一直使用es5的方式来写react。唉，人的一生，自身的努力很重要，但是也要考虑到历史的进程。。。于是我也就做了一点微小的工作，也开始采用es6的方式来写react了。

好了，说回正题，es5使用的是creatClass的方式来创建react组件，每一个函数都会自动被react绑定，所以就不会有“this.xxx is undefined.”这样的报错了。但是es6的class就不同了，你在<strong>render函数里使用到的this.xxx函数</strong>，都需要自己手动去构造函数中绑定它们，当然，你也可以通过插件来自动绑定，还有一种方式就是你可以在调用这个函数的地方，直接增加bind(this)来进行绑定，比如<code><input onChange={this.handleChange.bind(this)} /></code>这样来绑定this。

上面的几种方式都可以绑定this，不过在性能上来说，却还是有一定差距的。首先是引入第三方库来自动绑定，如果是一个比较大型的项目，引入一个比较小的库，来解决比较多比较繁琐的bind的问题，还是可以的，不过前提是项目比较大，需要在引入第三方库和人为绑定之间权衡利弊。剩下的方法就是在构造函数里绑定this还是在render函数里绑定this，工作量其实是差不多的，但是这里涉及到一个生命周期的问题，构造函数，在整个react声明周期里，只会调用一次，但是render就不一样了，任何组件的任何属性改变或者状态改变，都可能处罚render函数来重新渲染，所以render函数一般是会执行多次的，既然执行多次，那么里面的bind(this)自然也是会执行多次，这与在构造函数里只执行一次来比，当然就会慢一些。
有没有更好的办法呢？我在文章开始就说过了，es6有一个比较明显的特性就是箭头函数，它是会自动绑定this的，你可以在定义函数的地方直接使用箭头函数，代码如下：

```
handleChange = (event) => { 
  this.setState({
    text: event.target.value
  });
}
```

这样的话，也就不用考虑绑定this的事情了，是不是很方便啊。

#### 构造函数中的super

es6的constructor需要接收props，并且调用super(props)，这是相对于es5所改变的一点，这个记住就行了。

#### 初始化state

在es5中，react组件初始化的时候会执行一个getinitialState函数，而es6中却不是这样，在调用super之后，会直接设置一个this.state的对象来初始化，this.state = {...}。


#### getdefaultProps和propTypes

和初始化state一样，在es5中，也有一个函数来单独初始化props，就是getdefaultProps，而es6就是用上面的那个super方法，es5中可以直接在这个函数下面接着写propTypes来限定接收的参数的类型，而es6就不一样了，它将propTypes直接放在了react组件之外，因为这些将会是类本身的属性，所以需要放在类的外部来单独定义。es7易经又将定义属性改回了类的内部，不过这个还没普及。

```
class Person extends React.Component { 
  static propTypes = { 
    name: React.PropTypes.string, 
    age: React.PropTypes.string 
  }; 
  static defaultProps = { 
    name: '', 
    age: -1 
  }; 
  ...
}
```

#### 用哪个方法才好？

Facebook已经声明了React.createClass将最终会被ES6的classes替代，但是他们也说“在我们找到当前mixin所使用的例子的替代者以及在语言上支持类属性的初始化器前，我们不打算废弃React.createClass”。

在任何可以使用无状态函数式组件的地方使用它。简单而且会强制性地简化你的组件。

对于一些需要state，生命周期方法或是通过ref来操作潜在的DOM节点的复杂组件来讲，请使用class。

尽管知道这三种写法风格很棒，但在StackOverflow或者其他地方查找解决方案时可能仍会看到一些混杂着ES5和ES6写法的答案。ES6风格已经非常流行了，但这不会是你看到的唯一一种写法。




