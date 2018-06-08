---
title: react 16 新特性 说明
---

### context

说起context，就要从原生的react数据流说起。

在以前，react的数据传递，都是通过父组件的prop，向子组件传递，当组件嵌套层级过多时，就需要一层一层的将prop传递下去，当组件嵌套的层级足够多时。。。那我只能说，代码给你，你来吧。

这里就需要说一下context，在之前，官方是没有context api的，并且也一直不推荐使用，就像redux一样，如果你不知道该不该用，或者不熟悉该不该用，那就不要用。

说了这么久，我们先来介绍下context是个什么东西吧，刚才说到，react的数据传递，都是通过父组件的prop，向子组件传递，当组件嵌套层级过多时，就需要一层一层的将prop传递下去，当组件嵌套的层级足够多时，就很蛋疼，所以就要解决啊，解决的思想是，将所有的数据，变成一个store，并且将这个store的位置提到最高，高于任何组件的，所以这个context，也就是指的“上下文环境”，让一个树状组件上所有组件都能访问同一个共同的对象

具体如下代码：

```javascript
// 我们假设我们开发的组件叫App，为了给这个App加一个全局的一个数据仓库，在这外面再包一层Provider就好了，然后把你想要传递的数据，当作Provider的属性，这样就可以使数据变成一个全局的数据了。
<Provider store={store}>
	<App />
 </Provider>

// Provider.js 
// 而Provider，里面就是直接返回他的子组件就好了，并且还有一个函数来获取当前的store
getChildContext(){
	return {
    	store: this.props.store
    };
}
render() {
	return this.props.children;
}

// store.js
// 然后来看看这个store，这里的createStore是直接引用自redux，而这里的reducer，就是redux里面的reducer，需要自己接受state，action，然后根据你自己的业务需要，判断action.type，来返回新的state，且务必遵守纯函数的原则。
const initValues = {
    // 初始化state数据
};

const store = createStore(reducer, initValues);







```
























