---
title: redux 店铺装修
---

最近在看店铺装修相关的文档，很很很麻烦的一个项目，刚好里面用到了redux，我发现店铺装修简化一下的话，简直是redux的十分好的一个示例啊，比官网上简单的示例要难，又比难的简单，而且条理结构什么的又很清楚，挺不错的啊。

我先上图吧。
![](http://oatasl78l.bkt.clouddn.com/redux%20ex1.png)
图1
![](http://oatasl78l.bkt.clouddn.com/redux%20ex2.png)
图2
![](http://oatasl78l.bkt.clouddn.com/redux%20ex3.png)
图3
![](http://oatasl78l.bkt.clouddn.com/redux%20ex4.png)
图4
![](http://oatasl78l.bkt.clouddn.com/redux%20ex5.png)
图5
![](http://oatasl78l.bkt.clouddn.com/redux%20ex6.png)
图6
![](http://oatasl78l.bkt.clouddn.com/redux%20ex7.png)
图7
![](http://oatasl78l.bkt.clouddn.com/redux%20ex8.png)
图8

<!-- more -->

好吧，这图截的也是够大的，将就看看吧。。。由图1可以看到，页面一共分为四部分：左，中上，中下，右，也就是氛围四个组件：Left，Top，Center，Right。接下来我说一下交互，如图2所示；我在左边的输入框中输入东西，然后点击增加，可以看到，刚刚新增的数据立马在右边下面部分显示了出来，展示的是刚才新增的数据和一个删除按钮，如图3所示；若你点击展示的数据，那么在右边，也就是编辑组件中会出现一个输入框和编辑按钮，你可以在这里对刚刚点击的数据进行修改然后保存，如图4，图5所示；保存过后就可以看到中间的显示模块对数据进行了更新，如图6所示；接下来是中间上方的保存模块，你点击保存按钮，这个模块同样也能取到当前的数据，这里就用一个弹窗代替好了，如图7所示；最后就是删除按钮，点击后删掉当前这一行的数据，如图8所示。

好累，说了这么多，口都干了。其实吧，一两句话就可以说完的：在这四个组件中，每个组件都可以取到state，并且可以对state进行操作。说完了，是不是很简单啊。。。

我们先来看看index.js吧，也就是入口文件。

```
var App = React.createClass({
    render: function(){
        return (
            <div className="row">
                <div className="col-sm-4" style={{ textAlign: 'center' }}>
                    <LeftComponent />
                </div>
                <div className="col-sm-4" style={{ borderRight: '1px solid #999', minHeight: '600px', textAlign: 'center' }}>
                    <TopComponent/>
                    <CenterComponent/>
                </div>
                <div className="col-sm-4" style={{ textAlign: 'center' }}>
                    <RightComponent />
                </div>
            </div>            
            );            
```

很简单吧，就是分别放了四个Component组件而已，我们也不用管这些组件是父子关系还是兄弟关系，因为redux不用考虑这些。然后我们去看看action吧。

```
// action 
module.exports = {
	addModelAction: function(name){
	    return {
	        type: 'ADD_MODEL',
	        name: name
	    }
	},
	editModelAction: function(index, name) {
		return {
			type: 'EDIT_MODEL',
			index: index,
			name: name
		}
	},
	deleteModelAction: function(index) {
		return {
			type: 'DELETE_MODEL',
			index: index
		}
	},
	activeModelAction: function(index) {
		return {
			type: 'ACTIVE_MODEL',
			index: index
		}
	}
};
```

在action.js这个文件里，我们写了四个action函数，每个函数都返回一个对象作为action，是不是很简单，每个action函数中返回的对象里type都是定义好了的，然后根据传入的参数来配置其他返回参数；然后再来看看reducer。

```
// reducers
var modelReducer = function(state, action){
    switch(action.type){
        case 'ADD_MODEL':
            state.items.push({
                name: action.name
            });
            return state;
            break;
        case 'DELETE_MODEL':
        	state.items.splice(action.index, 1);
            return state;
            break;
        case 'EDIT_MODEL':
        	state.items[action.index].name = action.name;
            return state;
            break;
        case 'ACTIVE_MODEL':
            state.activeIndex =  action.index; 
            return state;
            break;
        default:
            return state;
    }
};

module.exports = modelReducer;
```
这个也不难，我们都知道reducer需要传入两个参数：state和action，通过action里的type来对state进行相应的操作，最后return state。

好了，reducer和action都看了，redux三要素现在来看看store。

```
import { createStore } from 'redux'
import modelReducer from './reducer'
var store = createStore(modelReducer, {items: [], activeIndex: -1 });
module.exports = store;
```
对的，一共就这几行，store=createStore(modelReducer, {items: [], activeIndex: -1 })，第一个参数是reducer，必需的，另一个是state初始化的值，非必需，若不传则默认为undefined。

其实到这里，这个示例就已经到了尾声了；剩下的就是这四个组件的内部布局，还有就是给按钮绑定事件，当点击的时候用dispatch分发一个对应的action函数给reducer，这些我在之前也已经写过了，所以这里就不再赘述了。

