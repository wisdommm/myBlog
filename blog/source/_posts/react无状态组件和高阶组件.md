---
title: react——无状态组件和高阶组件
---

#### 无状态组件

先来说说无状态组件吧，顾名思义，无状态组件就是指的没有状态的组件，众所周知在react中，状态指的就是state，而没有state，会有什么好处和坏处呢？

react中所有的数据，都是通过state和props来保存的，state主要是组件内部的数据交流，可以进行各种读写操作，而props主要是组件间的数据交流，一般针对props仅仅执行只读，不会做额外的操作。所以话说回来，无状态组件，没有state，那么内部的逻辑必将大大减少，编写react组件的方便性也将大大提高。

下面是一个简单的无状态组件：

```
function Hello(props{
  return <div>Hello{props.name}</div>
}
ReactDOM.render(<Hello name="world" />, mountNode)

// 最后一句也可以这么写
ReactDOM.render(Hello({"name":"world"}), mountNode)
```

通过这个简单的示例可以看到，原本需要写的<code>react</code>类定义（<code>React.createClass</code>或者<code>class Component extends React.Component</code>）来创建自己组件的定义，但是由于这仅仅是一个无状态组件（无状态函数），<code>react</code>在渲染的时候也省掉了奖<code>react</code>组件类实例化的过程。

所以对于一些纯静态展示的功能模块，可以考虑作为无状态组件，而无状态组件用来实现服务端渲染也是比较方便的，只要避免去获取dom节点就可以了。

<!-- more -->

#### 无状态组件的生命周期方法

我们可以看到，无状态组件就剩了一个<code>render</code>方法，因此也就没有实现组件的生命周期方法，例如<code>componentDidMount</code>, <code>componentWillUnmount</code>等。那应该怎么办呢？这个时候就用到了高阶组件。

#### 高阶组件

高阶组件是什么？通过函数，向现有的组件类添加逻辑，就叫做高阶组件。

之前上面已经说了，无状态组件是没有<code>state</code>的，所以里面基本也就没有什么逻辑，可是如果我现在需要添加逻辑了，且又想继续使用这个无状态组件，怎么办呢？那就可以使用高阶组件。

先来看看这个示例：

```
function noId() {
  return function(Comp) {
    return class NoID extends Component {
      render() {
        const {id, ...others} = this.props;
        return (
          <Comp {...others}/>
        )
      }
    }
  }
}

const WithoutID = noId()(Comp);
```

这个例子向我们展示了高阶组件的工作方式，通过函数和闭包等方法，改变已有组件的行为，示例里面改变了id属性。

之所以称之为高阶组件，是因为在react中，这种嵌套的逻辑关系会反映到组件树上，层层嵌套就像高阶函数的function in function一样，如下图所示：

![](http://oatasl78l.bkt.clouddn.com/%E9%AB%98%E9%98%B6%E7%BB%84%E4%BB%B6.png)<br />
从上图可以看出组件树虽然嵌套了许多层，但是实际渲染的无状态组件中的dom结构并没有发生改变，所以我们可以放心地使用高阶组件，哪怕重复使用，也不必担心影响输出的dom结构。而借助函数的表现能力，高阶组件的用途几乎是无穷尽的。

#### 适配器

有时候你需要替换一些已有的组件，而新组件接受的参数和原组件并不完全一样.

你可以考虑修改所有原组件的代码来保证传入正确的参数——so bad。

你也可以通过使用高阶组件，把新组件做一层封装：

```
class ListAdapter extends Component {
    mapProps(props) {
        return {/* new props */}
    }
    render() {
        return <NewList {...mapProps(this.props)} />
    }
}
```

如果有十个组件需要适配呢？当然，每个组件写十遍也不是不行，不过，高阶组件或许会给你一个更好的答案：

```
function mapProps(mapFn) {
    return function(Comp) {
        return class extends Component {
            render() {
                return <Comp {...mapFn(this.props)}/>
            }
        }
    } 
}

const ListAdapter = mapProps(mapPropsForNewList)(NewList);
```

借助高阶组件，关注点被分离得足够彻底，你不需要考虑组件的渲染，你只需要考虑属性的map就够了。

#### 处理副作用

在组件中，往往有很多的状态和副作用要处理，最常见的情况就是异步了。

假设我们需要异步加载一个用户列表，通常的代码会是这样的：

```
class UserList extends Component {
  constructor(props) {
    super();
    this.state = {
      list: []
    }
  }
  componentDidMount() {
    loadUsers()
      .then(data=> 
        this.setState({list: data.userList})
      )
  }
  render() {
    return (
      <List list={this.state.list} />
    )
  }
  /* other bussiness logics */
}
```

在实际情况中，以上代码往往还会和其他一些业务函数混杂在一起，如果再来一个或多个其他的列表呢，不仅代码会重复，大量有状态和副作用的组件也使得组件难以测试。或许你会考虑使用一些数据管理工具，比如flux或redux，是的，当然可以，可是这就像大炮打蚊子，杀鸡用牛刀。所以回到这个问题上来，我们只是想做一个异步的列表，仅此而已。

使用高阶函数试试：

```
function connectPromise({promiseLoader, mapResultToProps}) {
  return Comp=> {
    return class AsyncComponent extends Component {
      constructor(props) {
        super();
        this.state = {
          result: undefined
        }
      }
      componentDidMount() {
        promiseLoader()
          .then(result=> this.setState({result}))
      }
      render() {
        return (
          <Comp {...mapResultToProps(props)} {...this.props}/>
        )
      }
    }
  }
}


const UserList = connectPromise({
    promiseLoader: loadUsers,
    mapResultToProps: result=> ({list: result.userList})
})(List); //List can be a pure component
```

可以看到，这样不仅大量减少了重复的代码，还把散落在各处的异步逻辑封装进了可以单独管理和测试的函数中，在实际业务场景中，只需要按照“纯组件＋配置”就可以实现相同的功能，而无论是纯组件还是配置，都是对单元测试友好的，至少比异步组件友好多了吧。











































