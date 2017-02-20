---
title: react生命周期解读
---

前几天在工作中出现了一个比较怪异的bug，不过还好由于react的单向数据流，从最外层父组件一层一层地往里面找，在每一层都检查一下state和props的值是否正确，总会找到问题，只不过那个组件被前人嵌套了四五层，所以在定位问题的时候比较蛋疼，所以还是最好养成良好的习惯，组件嵌套层级不要太多，而且在开发的时候每一层就别忘了检测props和state，否则后来人会很累的。。。

经过了long long time，才在某一层组件中找到了问题，具体问题是父组件传入的props改变后子组件通过props来渲染的页面没有发生改变，有疑问吗？父组件传入的props改变，子组件的props不是会自动接收变化并且render吗？嗯，是的，为什么这里没有呢。。。原来有人在开发的时候在getInitialState函数中加了这么一句：this.state.xxx = this.props.xxx，通过这样，他就把props和state关联了起来，心中顿时有无数草泥马狂奔。。。怎么改呢？难道要我把和state关联了的props都取消关联？这个工作量不小。。。还好，react里有个监测props变化的函数：componentWillReceiveProps，然后我再在里面把更新后的props和state关联起来，让state等于新的props就好了，大功告成，这个怪异的问题就解决了。

有人会觉得奇怪吗？为什么在getInitialState函数中使用了this.state.xxx = this.props.xxx，而在当props更新后传进来时，state依然等于原来的props，这就说到了本片文章的主要，就是react的生命周期。

#### react组件的详细说明和生命周期

先来说一下react场见的函数吧。

<!-- more -->

1，render

重要的事情说三遍：render()方法是必须的！render()方法是必须的！！render()方法是必须的！！！

当使用render方法的时候，它最终return一个单子级组件，这个意思就是说返回<strong>一个</strong>dom 组件，比如一个 div 节点，也可以是其他自定义的组件，有个要点就是一定只能 return 一个 dom 组件，当你有多个单独的dom节点或者单独的自定义组件要 return 的时候，必须要在外面包裹一层 div 才行。当然，你也可以返回 null 或者 false 来表明不需要渲染任何东西。在 return 前会干什么呢？return 的作用是渲染页面的，而渲染页面需要数据react 里的数据主要就是 props 和 state，所以在 render 方法中，它先会自动监测 props 和 state，render 方法有几个需要注意的点：1，render 函数中不能修改 state，所以 setState 方法是不能用的；2，render 并不是渲染真是的 dom ，而是 Virtual DOM，所以在这个方法中，你并不能对 dom 进行操作，this.getDOMNode()也去不到任何节点，只能得到 null。3，不和浏览器交互，例如通过使用 setTimeout。

2，getInitialState

这是初始化state的方法，仅仅在组件挂载之前调用一次！

3，getDefaultProps

在组件创建的时候调用一次，然后返回值被缓存下来。如果父组件没有指定 props 中的某个键，则此处返回的对象中的相应属性将会合并到 this.props，（可以使用 in 来检测属性）。该方法在实例创建之前调用，因此不能依赖于 this.props。

4，componentWillMount

这个方法会在初始化渲染完毕之前立即调用，仅执行一次。可以使用 setState 方法。

5，componentDidMount

这个方法会在初始化渲染完毕之后立即调用，仅执行一次。这个时候已经渲染完毕了，所以 Virtual DOM 已经成为了真实的 DOM，你可以使用 this.getDOMNode()方法，发送 ajax 请求，或者 setTimeout 等方法。

6，componentWillReceiveProps

该方法仅仅在组件接收到新的 props 的时候调用。所以初始化的时候不会调用这个方法，传入的参数将是即将接收到的新的 props，一般会通过新老 props 比较，然后再决定需不需要更新页面。

7，shouldComponentUpdate

当组件接收到了新的 props 或者 state 的时候，即将要渲染页面之前调用这个方法。按常规来说，组件 state 或 props 更新后，往往都是需要重新渲染一下页面的，当然，这不是百分之百，所以这个时候就需要在渲染之前，在判断一下，是否真的需要渲染，在这个方法中可以获得 nextProps 和 nextState，然后我们可以比较，看是不是真的需要渲染，如果需要渲染，就return true，不需要渲染就 return false，这个方法默认是会返回 true 的。这个方法存在的意义就是不该渲染的时候不渲染，节约时间优化性能，当然，组件少了也没什么实际意义，可是当组件数量成百上千的时候，或许就会很明显地感觉到优化的性能了。

8，componentWillUpdate

这个方法主要是在接收到新的 props 或者 state 之前立刻调用，使用该方法做一些更新之前的准备工作，此时不能更改 state。

9，componentDidUpdate

个人觉得和componentDidMount类似，只是一个在初始化渲染后调用，一个在更新渲染后调用。

10，componentWillUnmount

在组件从 DOM 中移除的时候立刻被调用，在该方法中执行任何清理，比如无效的定时器，或者清除在 componentDidMount 中创建的 DOM 元素。

唉，好累，终于把这一大堆函数科普完了，我稍微归纳一下吧，方法大致有以下几类：初始化的时候执行的，初始化渲染前执行的，初始化渲染后执行的，接收更新时执行的，接收更新后渲染之前执行的，接收更新后渲染之后执行的，即将移除前执行的。这么多方法，都是和执行时间有关，并且一个方法后往往会执行另一系列的方法，这就是它的生命周期。

所以回到上面的问题，当父组件的 props 更新后，子组件的 props 也会更新没错，但是这时并不会重新去执行一次 getInitialState 方法，因为这个方法仅仅在初始化的时候调用一次，所以最后 render 时还是用的之前的 state 来进行渲染，所以我做的就是加上 componentWillReceiveProps 来进行人为更新 state。

#### 这篇文章到这里应该就可以结束了，不过我还有点意犹未尽，所以稍微多说一点吧。

![](http://oatasl78l.bkt.clouddn.com/react%20cycle1.png)

![](http://oatasl78l.bkt.clouddn.com/react%20cycle2.jpg)

上面两张图分别是各个不同的变化会产生的不同的一系列的方法调用，，以及每个方法是否能够对 state 进行更新都做了说明，可以看到，初始化的时候，props 变化的时候，state 变化的时候，都会触发不同方法来进行处理，有的函数是可以不写的，因为它默认会执行，经过上次改bug的事情后，我觉得吧，层级多了的时候还是最好在每个层级里面对 props 和 state 进行控制监测一下，以免发生不必要的错误。