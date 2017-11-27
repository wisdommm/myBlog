---
title: 从源码看react生命周期
---

React的主要思想，就是通过构建可复用的组件来构建页面，而每一个组件，就是一个有限的状态机，通过状态控制组件不同的渲染，从而得到不同的界面效果。每个组件都有生命周期，它规定了组件的一些自带方法需要在哪个阶段进行执行以及输出对应的结果。

有限状态机，表示有限个状态，以及在这些状态之间的数据转移和动作方法的执行的行为模型，一般通过状态、事件、转换和动作来描述有限状态机。例如页面上的都个东西有显示和隐藏两种情况，那么一般情况下我们会设计两个方法：show()和hide()来实现显示与隐藏的切换，而react只需要设置某一个状态为true或false，就可以实现这个组件的隐藏和显示转换了。
同时，react还引入了组件的生命周期这个概念，通过它就可以实现组件的状态机控制，从而达到“生命周期－状态－组件”的和谐画面。

#### 生命周期

之前一篇博客就已经说到了一下生命周期，我在这里再赘述一下吧，react组件自带的一些函数方法，比如getDefaultProps、getInitialState等，会在什么时候执行。

* 当首次装载组件时，按顺序执行 getDefaultProps、getInitialState、componentWillMount、render 和 componentDidMount；

* 当卸载组件时，执行 componentWillUnmount；

* 当重新装载组件时，此时按顺序执行 getInitialState、componentWillMount、render 和 componentDidMount，但并不执行 getDefaultProps；

* 当再次渲染组件时，组件接受到更新状态，此时按顺序执行 componentWillReceiveProps、shouldComponentUpdate、componentWillUpdate、render 和 componentDidUpdate.

然后具体的状态、属性改变，会触发什么方法，可以参考如下两张图，由于这篇博客重点不在于此，所以就不细细讲解了。<br>
![](http://oatasl78l.bkt.clouddn.com/react%20cycle1.png)<br>
![](http://oatasl78l.bkt.clouddn.com/react%20cycle2.jpg)<br>

疑问：

* 为什么react会按照这样的方式来，这样的顺序来执行生命周期？
* 为什么react多次render时，会执行生命周期的不同阶段。
* 为什么getDefaultProps只执行了一次？

<!-- more -->

#### react生命周期详解

react组件的生命周期，主要通过三种状态来管理，分别是<strong>MOUNTING、RECEIVE_PROPS、UNMOUNTING</strong>，而其他的方法，例如getDefaultProps、getInitialState，还包括自己定义的一些方法等，都是通过这三个状态来管理的，这三个状态会负责通知对应的组件当前所处的状态，应该执行生命周期中的哪个步骤，比如是否可以更新state，是否能调用自定义函数等。在这三种状态中，MOUNTING指的是加载组件，RECEIVE_PROPS指的是更新组件，UNMOUNTING指的是卸载组件，而每种情况，比如说加载组件（mountComponent），都会有两种方法，will和did，will指的是之前，比如说加载之前，更新之前，而did指的是之后，表示加载之后，更新之后。

#### creatClass创建组件

这是最react组件中的入口方法，负责管理生命周期中的getDefaultProps。getDefaultProps方法在react生命周期中只执行一次，那就是在首次加载组件初始化的时候执行。通过creatClass创建组件，利用原型继承ReactCompositeBase父类，然后按顺序合并mixins，设置初始化defaultProps，创建ReactElement元素，具体信息看看下面的源码。

```
// ReactCompositeComponent 的基类
var ReactCompositeComponentBase = function() {};

	// 将 Mixin 合并到 ReactCompositeComponentBase 的原型上
	assign(
	  ReactCompositeComponentBase.prototype,
	  ReactComponent.Mixin,
	  ReactOwner.Mixin,
	  ReactPropTransferer.Mixin,
	  ReactCompositeComponentMixin
	);

var ReactCompositeComponent = {
  LifeCycle: CompositeLifeCycle,
  Base: ReactCompositeComponentBase,

  // 创建组件
  createClass: function(spec) {
    // 构造函数
    var Constructor = function(props, context) {
      this.props = props;
      this.context = context;
      this.state = null;
      var initialState = this.getInitialState ? this.getInitialState() : null;
      this.state = initialState;
    };

    // 原型继承父类
    Constructor.prototype = new ReactCompositeComponentBase();
    Constructor.prototype.constructor = Constructor;

    // 合并 mixins
    injectedMixins.forEach(
      mixSpecIntoComponent.bind(null, Constructor)
    );
    mixSpecIntoComponent(Constructor, spec);

    // mixins 合并后装载 defaultProps (React整个生命周期中 getDefaultProps 只执行一次)
    if (Constructor.getDefaultProps) {
      Constructor.defaultProps = Constructor.getDefaultProps();
    }

    for (var methodName in ReactCompositeComponentInterface) {
      if (!Constructor.prototype[methodName]) {
        Constructor.prototype[methodName] = null;
      }
    }

    return ReactElement.createFactory(Constructor);
  }
}
```

所以这就解释了，为什么getDefaultProps会在react组件中很早的就被调用，且只被调用一次，因为getDefaultProps是通过Constructor来进行管理的，所以是在整个生命周期中最先开始执行。

#### 状态一：MOUNTING

mountComponent负责管理生命周期中的getInitialState、componentWillMount、render、componentDidMount方法。

mountComponent主要做了以下一些事。

首先通过mountComponent装载组件，此时将状态改为MOUNTING，通过geiInitalState获取初始化的state，初始化更新队列为null。

然后会判断componentWillMount是否存在，如果存在，则执行componentWillMount，若此时使用了setState方法，则不会出发re-render，而是会合并更新队列，和初始化的render一起执行。

到此时，已经完成了MOUNTING的工作，将更新状态改为NULL，如果后面再使用setState方法，则会更改更新状态，然后出发render函数重新渲染。

mountComponent本质上是通过递归的方法来渲染的内容，所以如果一个组件中包含了另外的子组件的话，怎会发现，父组件中的componentWillMount总是会在子组件中的componentWillMount之前调用，而父组件的componentDidMount总是会在子组件的componentDidMount后面调用。

最后当初始化渲染完成后，若存在componentDidMount，则出触发。

这段流程的源码如下：

```
// 装载组件
mountComponent: function(rootID, transaction, mountDepth) {
  // 当前状态为 MOUNTING
  this._compositeLifeCycleState = CompositeLifeCycle.MOUNTING;

  // 当前元素对应的上下文
  this.context = this._processContext(this._currentElement._context);

  // 当前元素对应的 props
  this.props = this._processProps(this.props);

  // 获取初始化 state
  this.state = this.getInitialState();

  // 初始化更新队列
  this._pendingState = null;
  this._pendingForceUpdate = false;

  // componentWillMount 调用setstate，不会触发rerender而是自动提前合并
  if (this.componentWillMount) {
    this.componentWillMount();
    if (this._pendingState) {
      this.state = this._pendingState;
      this._pendingState = null;
    }
  }

  // 得到 _currentElement 对应的 component 类实例
  this._renderedComponent = instantiateReactComponent(
    this._renderValidatedComponent(),
    this._currentElement.type
  );

  // 完成 MOUNTING，更新 state
  this._compositeLifeCycleState = null;

  // render 递归渲染
  var markup = this._renderedComponent.mountComponent(
    rootID,
    transaction,
    mountDepth + 1
  );

  // 如果存在 this.componentDidMount，则渲染完成后触发
  if (this.componentDidMount) {
    transaction.getReactMountReady().enqueue(this.componentDidMount, this);
  }

  return markup;
}
```

详细流程图如下：
![](http://oatasl78l.bkt.clouddn.com/mountComponent.jpeg)

#### 状态二：RECEIVE_PROPS

updateComponent负责管理react生命周期中的componentWillReceiveProps、shouldComponentUpdate、componentWillUpadte、render、compoentDidUpdate等方法。

在有组件更新时，react会通过updateComponent来更新组件，如果前后的元素不一致，则说明需要进行组件更新，此时会将状态改为RECEIVE_PROPS。

在进行组件更新的时候，react首先会检查有没有componentWillReceiveProps，如果有，则执行此函数，且componentWillReceiveProps函数中调用setState是不会重复触发render函数的，而是会进行render合并，到这个时刻，RECEIVE_PROPS的工作就已经做完了，然后将状态改为NULL，同时state也将进行更新操作，此时调用this.state，可以获得更新后的state。

然后会调用shouldComponentUpdate来作为最后的判断，判断组件是否更新，return true则会更新，否则不更新；之前已经说过了，react里的组件是通过递归来更新渲染的，而由于递归的特性，所以父组件的componentWillUpdate会在子组件componentWillUpdate之前执行，父组件的componentDidUpdate会在子组件componentDidUpdate之后执行。注意：禁止在shouldComponentUpdate和componentWillUpdate中调用setState，会造成循环调用，直至耗光浏览器内存后崩溃。

updateComponent的具体流程图如下：
![](http://oatasl78l.bkt.clouddn.com/upCom.jpeg)

下面是具体源码：

```
// 更新组件
updateComponent: function(transaction, prevParentElement, nextParentElement) {
  var prevContext = this.context;
  var prevProps = this.props;
  var nextContext = prevContext;
  var nextProps = prevProps;

  if (prevParentElement !== nextParentElement) {
    nextContext = this._processContext(nextParentElement._context);
    nextProps = this._processProps(nextParentElement.props);
    // 当前状态为 RECEIVING_PROPS
    this._compositeLifeCycleState = CompositeLifeCycle.RECEIVING_PROPS;

    // 如果存在 componentWillReceiveProps，则执行
    if (this.componentWillReceiveProps) {
      this.componentWillReceiveProps(nextProps, nextContext);
    }
  }

  // 设置状态为 null，更新 state
  this._compositeLifeCycleState = null;
  var nextState = this._pendingState || this.state;
  this._pendingState = null;
  var shouldUpdate =
    this._pendingForceUpdate ||
    !this.shouldComponentUpdate ||
    this.shouldComponentUpdate(nextProps, nextState, nextContext);
  if (!shouldUpdate) {
    // 如果确定组件不更新，仍然要设置 props 和 state
    this._currentElement = nextParentElement;
    this.props = nextProps;
    this.state = nextState;
    this.context = nextContext;
    this._owner = nextParentElement._owner;
    return;
  }
  this._pendingForceUpdate = false;

  ......

  // 如果存在 componentWillUpdate，则触发
  if (this.componentWillUpdate) {
    this.componentWillUpdate(nextProps, nextState, nextContext);
  }

  // render 递归渲染
  var nextMarkup = this._renderedComponent.mountComponent(
    thisID,
    transaction,
    this._mountDepth + 1
  );

  // 如果存在 componentDidUpdate，则触发
  if (this.componentDidUpdate) {
    transaction.getReactMountReady().enqueue(
      this.componentDidUpdate.bind(this, prevProps, prevState, prevContext),
      this
    );
  }
},
```

#### 状态三：UNMOUNTING

unmountComponent 负责管理生命周期中的 componentWillUnmount。

首先将状态设置为 UNMOUNTING，若存在 componentWillUnmount，则执行；如果此时在 componentWillUnmount 中调用 setState，是不会触发 reRender。更新状态为 NULL，完成组件卸载操作。

具体代码如下所示：

```
// 卸载组件
unmountComponent: function() {
  // 设置状态为 UNMOUNTING
  this._compositeLifeCycleState = CompositeLifeCycle.UNMOUNTING;

  // 如果存在 componentWillUnmount，则触发
  if (this.componentWillUnmount) {
    this.componentWillUnmount();
  }

  // 更新状态为 null
  this._compositeLifeCycleState = null;
  this._renderedComponent.unmountComponent();
  this._renderedComponent = null;

  ReactComponent.Mixin.unmountComponent.call(this);
}
```

#### setState更新机制

说起setState，想必用过react的都会知道吧，不过setState的更新机制，你真的知道吗？

当你调用setState的时候，会对state以及_pendingState更新队列进行合并操作，但是，真正更新state的幕后黑手其实是replaceState。replaceState会先判断当前状态是否是MOUNTING，如果不是，则会立即调用ReactUpdates.enqueueUpdate来执行更新。

当状态不为MOUNTING和RECEIVEPROPS的时候，performUpdateIfNecessary 会获取 _pendingElement、_pendingState、_pendingForceUpdate，并调用 updateComponent 进行组件更新。

如果在 shouldComponentUpdate 或 componentWillUpdate 中调用 setState，此时的状态已经从 RECEIVING_PROPS -> NULL，则 performUpdateIfNecessary 就会调用 updateComponent 进行组件更新，但 updateComponent 又会调用 shouldComponentUpdate 和 componentWillUpdate，因此造成循环调用，使得浏览器内存占满后崩溃。产生如下图的这种死循环：

![](http://oatasl78l.bkt.clouddn.com/sta.jpeg)

setState具体源码如下所示：

```
// 更新 state
setState: function(partialState, callback) {
  // 合并 _pendingState
  this.replaceState(
    assign({}, this._pendingState || this.state, partialState),
    callback
  );
},

// 更新 state
replaceState: function(completeState, callback) {
  validateLifeCycleOnReplaceState(this);

  // 更新队列
  this._pendingState = completeState;

  // 判断状态是否为 MOUNTING，如果不是，即可执行更新
  if (this._compositeLifeCycleState !== CompositeLifeCycle.MOUNTING) {
    ReactUpdates.enqueueUpdate(this, callback);
  }
},

// 如果存在 _pendingElement、_pendingState、_pendingForceUpdate，则更新组件
performUpdateIfNecessary: function(transaction) {
  var compositeLifeCycleState = this._compositeLifeCycleState;

  // 当状态为 MOUNTING 或 RECEIVING_PROPS时，则不更新
  if (compositeLifeCycleState === CompositeLifeCycle.MOUNTING ||
      compositeLifeCycleState === CompositeLifeCycle.RECEIVING_PROPS) {
    return;
  }

  var prevElement = this._currentElement;
  var nextElement = prevElement;
  if (this._pendingElement != null) {
    nextElement = this._pendingElement;
    this._pendingElement = null;
  }

  // 调用 updateComponent
  this.updateComponent(
    transaction,
    prevElement,
    nextElement
  );
}
```

总结：

* React 通过三种状态：MOUNTING、RECEIVE_PROPS、UNMOUNTING，管理整个生命周期的执行顺序；

* setState 会先进行 _pendingState 更新队列的合并操作，不会立刻 reRender，因此是异步操作，且通过判断状态（MOUNTING、RECEIVE_PROPS）来控制 reRender 的时机；

* 不建议在 getDefaultProps、getInitialState、shouldComponentUpdate、componentWillUpdate、render 和 componentWillUnmount 中调用 setState，特别注意：不能在 shouldComponentUpdate 和 componentWillUpdate 中调用 setState，会导致循环调用。













