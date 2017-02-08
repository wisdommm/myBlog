---
title: redux example(1)
---

最近在学习redux，看了不少相关的资料，发现这个[官方教程](http://cn.redux.js.org/docs/introduction/Examples.html)的一些例子还是很不错的，由浅入深各个档次都有，所以我稍微看了下，准备有时间的话将示例都自己弄懂，然后分别写篇详细的解读出来，由于是临时起意，所以今天先写第一篇吧。

#### counter-vanilla

这是redux中的第一个示例，也就是说是最简单的一个例子，不多说了，先看看代码吧。

<!-- more -->

```
<!DOCTYPE html>
<html>
  <head>
    <title>Redux basic example</title>
    <script src="https://npmcdn.com/redux@latest/dist/redux.min.js"></script>
  </head>
  <body>
    <div>
      <p>
        Clicked: <span id="value">0</span> times
        <button id="increment">+</button>
        <button id="decrement">-</button>
        <button id="incrementIfOdd">Increment if odd</button>
        <button id="incrementAsync">Increment async</button>
      </p>
    </div>
    <script>
      function counter(state, action) {
        if (typeof state === 'undefined') {
          return 0
        }

        switch (action.type) {
          case 'INCREMENT':
            return state + 1
          case 'DECREMENT':
            return state - 1
          default:
            return state
        }
      }

      var store = Redux.createStore(counter)
      var valueEl = document.getElementById('value')
      function render() {
        valueEl.innerHTML = store.getState().toString()
      }

      render()
      store.subscribe(render)

      document.getElementById('increment')
        .addEventListener('click', function () {
          store.dispatch({ type: 'INCREMENT' })
        })

      document.getElementById('decrement')
        .addEventListener('click', function () {
          store.dispatch({ type: 'DECREMENT' })
        })

      document.getElementById('incrementIfOdd')
        .addEventListener('click', function () {
          if (store.getState() % 2 !== 0) {
            store.dispatch({ type: 'INCREMENT' })
          }
        })

      document.getElementById('incrementAsync')
        .addEventListener('click', function () {
          setTimeout(function () {
            store.dispatch({ type: 'INCREMENT' })
          }, 1000)
        })
    </script>
  </body>
</html>      
```

如上面代码所示，一共也就几十行，而且就这一个文件，毕竟是最基础最简单的一个示例，所以也没有什么文件结构和包依赖，以及到node服务器等，，唯一的一个依赖也通过了script标签引入。
不废话了，下面开始讲解一下代码吧。

dom结构就很简单吧，主要就是一个span标签包裹一个数字，然后就是几个button按钮，点击按钮的时候会让span标签里的数字增一、减一、奇数加一变为偶数、异步加一（也就是加个定时器延迟）。

下面来看script标签里的代码，也就是实现逻辑和功能的主要代码。

```
function counter(state, action) {
	if (typeof state === 'undefined') {
		return 0
	}
	switch (action.type) {
		case 'INCREMENT':
			return state + 1
		case 'DECREMENT':
        	return state - 1
       	default:
        	return state
	}
}
```

这段代码主要实现的是reducer，它主要接收两个参数，一个是state，另一个是action，state指代的是react中的state（状态），也就是这里span标签中的数字，action是一个对象，通过action，reducer就知道该做什么，比如这段代码就是通过判断action.type，来对state进行操作。

这里有一个值得注意的点，那就是给state设置一个return的默认值，一般来说就写成<strong>function counter(state = initialState, action)</strong>，es6的写法，意思在忘了传state的时候给state传入一个默认值。

var store = Redux.createStore(counter)

上面这句代码，主要实现的是用createStore方法，创建store，而传入的参数counter我上面已经说过了，是一个reducer，所以store的创建实际上是这样的：var store = createStore(reducer)。

<strong>整个应用，只能有一个store！</strong>可是，reducer却可以有多个，这样问题就来了，由于创建store的时候需要传入reducer，而store只能有一个，但是reducer却可以有多个，那怎么办？[]react-redux](https://github.com/reactjs/react-redux)中又一个管理reducer的方法，叫做<strong>combineReducers</strong>，它能把所有的reducer集中整合为一个reducers，其使用方法是：

```
var reducers = combineReducers({
  reducers1,
  reducers2...
})
```

关于react-redux的具体细节后面再说。


```
var valueEl = document.getElementById('value')
function render() {
	valueEl.innerHTML = store.getState().toString()
}
render()
store.subscribe(render)
```

这段代码就主要就是执行render函数了，store.getState() 函数，它将会返回state。如果是常规react的render函数的话，是将一个dom节点挂载到root element下面去，而在这个示例中，只要将状态挂载到对应的节点上去就可以了。最后一行代码是注册状态监听器，在这里指当store里面的state发生改变时，触发render回调函数


```
document.getElementById('increment').addEventListener('click', function () {
          store.dispatch({ type: 'INCREMENT' })
        })
document.getElementById('decrement').addEventListener('click', function () {
          store.dispatch({ type: 'DECREMENT' })
        })
document.getElementById('incrementIfOdd').addEventListener('click', function () {
          if (store.getState() % 2 !== 0) {
            store.dispatch({ type: 'INCREMENT' })
          }
        })
document.getElementById('incrementAsync').addEventListener('click', function () {
          setTimeout(function () {
            store.dispatch({ type: 'INCREMENT' })
          }, 1000)
        })
```

这段代码主要是给下面的各个按钮绑定事件，这里需要做的是分发action，通过<strong>store.dispatch(action)</strong>这个方法，把action传递给reducer，而reducer中已经定义好了具体的函数，会根据接收到的action来判断对state进行什么操作。

好了，第一个示例到此就基本结束了，下面我再好好地整理一下redux的具体流程：

1，通过createStore方法创建store，传入reducer参数，<strong>且一个应用只能有一个store。</strong>store常用的方法有：getState方法，用来获取state；dispatch方法用来分发action，subscribe方法用来注册监听器执行回调函数。

2，创建好store后，通过dispatch方法来分发action给reducer，action是一个对象。

3，reducer接收到了state（state可传可不传，但是要在reducer中写上没有传state时的情况判断），action后，会根据action里的type属性来对state进行对应的操作。

<strong>我描绘一下redux的流程图，大致这样的：const store = createStore(reducer) --> store.dispatch(action) --> reducer(state, action) --> 根据state渲染页面</strong>


