---
title: redux源码解读
---

这是2017年的第一篇博客，上周参加完各种年会聚会，本来准备周末写的，可是周末又突然有事，所以一直拖到现在，而且这是2017的第一篇技术博客，所谓好的开始是成功的一半，希望新的一年，自己可以继续坚持，变得更好。

### redux

其实我工作项目中接触的redux并不多，应该说只有一两个项目用到了redux，要求也是基本的redux的使用，dispatch一个action这种程度，react-redux也还没有涉及到，可能是项目的复杂程度不够大吧，所以涉及到redux的相关技术现在都是靠自学（唉，现在公司PHP迁移到JAVA，业务压力这么重，哪有时间自学啊，连示例都没看完，囧2333～）。

吐槽完毕，回到正题上来吧，学习一个库和框架最好的方法就是直接看源码，看源码好处有很多，不仅仅是学习API，还可以学习别人编写这个库（框架）的思想，这才是最为重要的，所以这次来写一篇redux源码相关的博客。

### redux源码解读

先把redux下载下来，发现很小，且没有依赖其他第三方库，简直是短小精悍啊，下面就让我好好见识下里面的思想。

我们直接看到src文件夹下，发现下面一共有七个文件或文件夹，分别是utils，applyMiddleware.js，bindActionCreators.js，combineReducers.js，compose.js，createStore.js，index.js。我们分别来看看。

<!-- more -->

#### utils

这个是公用的工具方法，里面就warning.js一个文件，打开文件可以看到，里面就一个warning函数。

```
export default function warning(message) {
  if (typeof console !== 'undefined' && typeof console.error === 'function') {
    console.error(message)
  }
  try {
    throw new Error(message)
  } catch (e) { }
}
```

这个函数的主要目的就是抛出异常（错误），这个应该没什么好说的。

#### index.js

接下来看看index.js文件，这种文件的主要目的一般是作为入口文件，里面代码也不多。

```
function isCrushed() {}

if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning(
    'You are currently using minified code outside of NODE_ENV === \'production\'. ' +
    'This means that you are running a slower development build of Redux. ' +
    'You can use loose-envify (https://github.com/zertosh/loose-envify) for browserify ' +
    'or DefinePlugin for webpack (http://stackoverflow.com/questions/30030031) ' +
    'to ensure you have the correct code for your production build.'
  )
}

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose
}
```

首先看到一个空函数<code>isCrushed</code>，后面跟了一个<code>if</code>判断，第一个判断条件是<code>process.env.NODE_ENV !== 'production'</code>，我记得process是node里的一个核心模块，然后我去node环境找了下process.env.NODE_ENV，发现undefined，说明本身是没有这个参数的，谷歌了一下，发现这是node的环境变量，如果<code>process.env.NODE_ENV == 'production'</code>，则表示生产环境，[具体参考地址在这里](https://cnodejs.org/topic/53fec0d97c1e2284788983d6)。这是第一个判断条件，而后面两个判断条件都是判断文件代码是否被压缩，因为如果被压缩了，那么isCrushed函数就不叫isCrushed函数了，同时浏览器发出警告。最后export一堆接口，分别是createStore,combineReducers,bindActionCreators,applyMiddleware,compose这五个。

#### createStore

这应该是redux主要的一个函数，不对，应该是最主要的，顾名思义，<code>createStore</code>的意思就是创建store，而一个应用中只有唯一的一个store，你说这能不重要吗？

废话不多说，先看代码吧。

```
// 首先就定义了一个 ActionTypes 对象，它是一个action，是一个Redux的私有action，不允许外界触发，你可以在这段代码的最后面发现这个action就是初始化时，dispatch的action，也就是初始化Store的状态树和改变reducers后初始化Store的状态树。

export var ActionTypes = {
	INIT: '@@redux/INIT'
}

// 这是主函数，createStore，它能够接收三个参数，reducer是必须的，preloadedState和enhancer是不必须，先来讲讲参数吧，reducer就是redux里的全局reducer，相当于一个全局的回调函数，返回下一个状态，接受两个参数：当前状态和触发的action（我自己编的，2333～）；preloadedState表示的是初始时的state，比如服务端渲染得到的初始状态，但是如果使用combineReducers来生成reducer，那必须保持状态对象的key和combineReducers中的key相对应；而enhancer是store的增强器函数，可以指定为第三方的中间件，时间旅行，持久化等等，但是这个函数只能用redux提供的applyMiddleware函数来生成。（这个我不怎么清楚，也没怎么用过。。。）

export default function createStore(reducer, preloadedState, enhancer) {

	// 这是判断是否有人传入参数顺序不对，如果是的话调整一下参数位置。

  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  // enhancer只允许传入函数

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }

  // reducer是必填的

  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

  // 初始化参数，将当前状态currentState设置为初始状态preloadedState，初始化时间监听器数组等。

  var currentReducer = reducer
  var currentState = preloadedState
  var currentListeners = []
  var nextListeners = currentListeners
  var isDispatching = false

  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  // getState方法，返回当前的状态树currentState。

  function getState() {
    return currentState
  }

  // subscribe函数，是给状态树添加监听函数，每当调用dispatch的时候，所有的监听函数就会执行。

  function subscribe(listener) {

  	// 判断传入参数是否为函数

    if (typeof listener !== 'function') {
      throw new Error('Expected listener to be a function.')
    }

    var isSubscribed = true

    // nextListeners是当前监听的事件列表，每次调用subscribe的时候，就会向nextListeners中push一个被监听的函数，并且将isSubscribed赋值为true。

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    // 调用subscribe函数的时候，就会返回一个unsubscribe函数，用来取消监听函数，unsubscribe函数会先将isSubscribed的值改为false，同时将这个函数从当前监听的事件列表，也就是nextListeners中去除掉。

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      var index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }

  // dispatch函数，作用是分发action

  function dispatch(action) {

  	// 首先判断action格式
    
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
      )
    }

    // action中type参数是必需的

    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
      )
    }

    // 判断是否正在dispatch

    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    // 开始dispatch，将isDispatching赋值为true，然后将currentState和action传给reducer，完成后再将isDispatching的值改为false。

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    // dispatch完了后执行被监听了的函数，遍历函数列表nextListeners，分别执行里面的每个函数，最后会返回这个action。

    var listeners = currentListeners = nextListeners
    for (var i = 0; i < listeners.length; i++) {
      listeners[i]()
    }

    return action
  }

  // 更改reducer，传入一个新的reducer，并且通过分发初始action，来初始化替换后reducer生成的初始化状态并且赋予store的状态。

  function replaceReducer(nextReducer) {

  	// 容错判断，reducer必须是函数，是的话就用nextReducer替换掉currentReducer，再重新dispatch一下初始action，初始store。
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.INIT })
  }

  function observable() {
    var outerSubscribe = subscribe
    return {
      subscribe(observer) {
        if (typeof observer !== 'object') {
          throw new TypeError('Expected the observer to be an object.')
        }

        function observeState() {
          if (observer.next) {
            observer.next(getState())
          }
        }

        observeState()
        var unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }

  dispatch({ type: ActionTypes.INIT })

  // 最后就是返回这些所有的函数，作为暴露的接口使用。

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}

```

终于把createStore文件解读完了，下面来看看combineReducers函数。

#### combineReducers.js

reducer函数的作用是得到action后生成新的state，而由于一个app只拥有一个state，所以这个state肯定是比较复杂，数据比较多的，因此这个对应的处理生成state的reducer肯定也会是很复杂的，所以为了降低复杂度，也为了逻辑的解耦，我们会把这个大的reducer拆分成多个小的reducer，而这个combineReducers函数是为了整合所有的小的reducer，重新变为一个大的reducer，这就是combineReducers存在的原因，本来我想直接就把combineReducers的源码放上来的，不过我觉得可以先不忙，先<strong>具体</strong>的说一说combineReducers做的事情，下面这段讲解借鉴于[阮一峰的博客](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)。

```
// 可以看到，这个方法有一个前提，就是state的属性名必须与子reducer同名

const reducer = combineReducers({
  doSomethingWithA,
  processB,
  c
})

// 如果不同名，则如下

const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})

// 等同于

function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}

// 然后下面是一个ombineReducers函数的简单的实现，看完了这个再看源码吧。

const combineReducers = reducers => {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce(
      (nextState, key) => {
        nextState[key] = reducers[key](state[key], action);
        return nextState;
      },
      {} 
    );
  };
};

// 其实这个简洁版的combineReducers还是比较容易理解的，主要就是先拿到reducers的key，然后通过reduce函数将原reducer的各个函数遍历一下返回一个新的reducers函数，很简单吧，至少看到这里，combineReducers函数的大概作用都是已经明白了吧，下面我门来看看其源码。

// 前面这三个函数是对各个参数的合法性进行校验，比如reducer，state等。
function getUndefinedStateErrorMessage(key, action) {
  var actionType = action && action.type
  var actionName = actionType && `"${actionType.toString()}"` || 'an action'

  return (
    `Given action ${actionName}, reducer "${key}" returned undefined. ` +
    `To ignore an action, you must explicitly return the previous state.`
  )
}

function getUnexpectedStateShapeWarningMessage(inputState, reducers, action, unexpectedKeyCache) {
  var reducerKeys = Object.keys(reducers)
  var argumentName = action && action.type === ActionTypes.INIT ?
    'preloadedState argument passed to createStore' :
    'previous state received by the reducer'

  if (reducerKeys.length === 0) {
    return (
      'Store does not have a valid reducer. Make sure the argument passed ' +
      'to combineReducers is an object whose values are reducers.'
    )
  }

  if (!isPlainObject(inputState)) {
    return (
      `The ${argumentName} has unexpected type of "` +
      ({}).toString.call(inputState).match(/\s([a-z|A-Z]+)/)[1] +
      `". Expected argument to be an object with the following ` +
      `keys: "${reducerKeys.join('", "')}"`
    )
  }

  var unexpectedKeys = Object.keys(inputState).filter(key =>
    !reducers.hasOwnProperty(key) &&
    !unexpectedKeyCache[key]
  )

  unexpectedKeys.forEach(key => {
    unexpectedKeyCache[key] = true
  })

  if (unexpectedKeys.length > 0) {
    return (
      `Unexpected ${unexpectedKeys.length > 1 ? 'keys' : 'key'} ` +
      `"${unexpectedKeys.join('", "')}" found in ${argumentName}. ` +
      `Expected to find one of the known reducer keys instead: ` +
      `"${reducerKeys.join('", "')}". Unexpected keys will be ignored.`
    )
  }
}

function assertReducerSanity(reducers) {
  Object.keys(reducers).forEach(key => {
    var reducer = reducers[key]
    var initialState = reducer(undefined, { type: ActionTypes.INIT })

    if (typeof initialState === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined during initialization. ` +
        `If the state passed to the reducer is undefined, you must ` +
        `explicitly return the initial state. The initial state may ` +
        `not be undefined.`
      )
    }

    var type = '@@redux/PROBE_UNKNOWN_ACTION_' + Math.random().toString(36).substring(7).split('').join('.')
    if (typeof reducer(undefined, { type }) === 'undefined') {
      throw new Error(
        `Reducer "${key}" returned undefined when probed with a random type. ` +
        `Don't try to handle ${ActionTypes.INIT} or other actions in "redux/*" ` +
        `namespace. They are considered private. Instead, you must return the ` +
        `current state for any unknown actions, unless it is undefined, ` +
        `in which case you must return the initial state, regardless of the ` +
        `action type. The initial state may not be undefined.`
      )
    }
  })
}

// 下面正式进入combineReducers.js文件的主函数，combineReducers（这句话怎么这么怪）

export default function combineReducers(reducers) {
  
  // 可以看到，首先是获取老的reducers的key数组
  
  var reducerKeys = Object.keys(reducers)
  var finalReducers = {}
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i]

    // 判断老的reducer传入的key所对应的value是否是function，如果是，将老的reducer中的对象放入新的finalReducers对象中。
    
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  var finalReducerKeys = Object.keys(finalReducers)
  
  // 判断node环境是否为生产环境，之前已经说过了，后面就不再赘述了。

  if (process.env.NODE_ENV !== 'production') {
    var unexpectedKeyCache = {}
  }
  // 验证finalReducers是否合法。
  var sanityError
  try {
    assertReducerSanity(finalReducers)
  } catch (e) {
    sanityError = e
  }

  // 返回最终生成的reducer。
  return function combination(state = {}, action) {
    if (sanityError) {
      throw sanityError
    }

    if (process.env.NODE_ENV !== 'production') {
      var warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache)
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    var hasChanged = false
    var nextState = {}
    for (var i = 0; i < finalReducerKeys.length; i++) {
      var key = finalReducerKeys[i]
      var reducer = finalReducers[key]
      var previousStateForKey = state[key]
      var nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        var errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }

    // 遍历一遍看是否改变，然后返回原有状态值或者新的状态值
    
    return hasChanged ? nextState : state
  }
}
```

#### compose.js

```
// compose可以接受一组函数参数，从右到左来组合多个函数，然后返回一个组合函数，reduceRight和reduce的差异就是reduceRight是从右边开始遍历的，至于后面的逗号简写方式，可以参考[这个问题](https://segmentfault.com/q/1010000007320321)。

export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  const last = funcs[funcs.length - 1]
  const rest = funcs.slice(0, -1)
  return (...args) => rest.reduceRight((composed, f) => f(composed), last(...args))
}
```

### 总结？

这一次的redux源码解读先讲到这里吧，感觉这个信息量已经很大了，这些代码都是比较精简且耦合度低，没依赖其他的第三方库，仅仅读一遍还是远远不够的，除了代码的优秀，作者的设计思想也是十分优秀的，我还需要不断的学习才能达到这个高度，而至于bindActionCreators和applyMiddleware两个文件没有讲，我准备之后再来讲，外面天都黑了，非工作日一个人在办公室坐了一天，感觉周围都是寒冷😓。。。

2017年的第一篇博客over。