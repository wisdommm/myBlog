---
title: vue todo示例
---

最近抽空看了下vue，一个轻量级的前端框架（类库），它采用的mvvm模式，且数据双向绑定，刚好官方网站上有一个todo实例，我感觉和redux的todo实例很类似，所以准备将这两个框架的todo实例分别解读一下，来比较一下个中异同。

这个代码就两部分，先是html：
```
<div id="app">
  <input v-model="newTodo" v-on:keyup.enter="addTodo">
  <ul>
    <li v-for="todo in todos">
      <span>{{ todo.text }}</span>
      <button v-on:click="removeTodo($index)">X</button>
    </li>
  </ul>
</div>
```

<!-- more -->

接下来是js代码：

```
new Vue({
  el: '#app',
  data: {
    newTodo: '',
    todos: [
      { text: 'Add some todos' }
    ]
  },
  methods: {
    addTodo: function () {
      var text = this.newTodo.trim()
      if (text) {
        this.todos.push({ text: text })
        this.newTodo = ''
      }
    },
    removeTodo: function (index) {
      this.todos.splice(index, 1)
    }
  }
})
```

我第一次看到这段代码的时候觉得有点诧异，就这么点代码就实现了一个todo list，比react少了好多，如果是redux的todo list，别说代码，文件夹数量就可以让你一双手数不过来，而这vue两个文件夹一共才这么点代码，下面来仔细看看这个代码吧。
先看下html代码吧，除了原生的html代码外，这里有几个vue特有的属性，显而易见，所有以<strong>v</strong>开头的属性，都是vue特有的属性，比如这里的v-model="newTodo"，v-on:keyup.enter="addTodo"，v-for="todo in todos"，v-on:click="removeTodo($index)"，一共就这四个吧，而{{ todo.text }}就是字符串模版了，从js文件的某个对象中获取数据来填充到页面上。
js代码也很少，估计一共才十多行，抛去中间的内容不看，就成了new Vue({})了，很显然这就是new了一个Vue实例，算是初始化，然后我们再来对内容一行一行地解读。

```
el: '#app'
```

先看一下app在html文件里指代的是什么，一看便知，app在这里指代的是最外层div的id，而这里的＃应该类似于jQuery选择器中的符号吧（所以我猜测这里也可以用“.app”来选择class为app的元素来作为Vue的容器），前面的el应该指的是element吧。
再看下面：

```
data: {
    newTodo: '',
    todos: [
      { text: 'Add some todos' }
    ]
  }
```

这个data很明显说明了这里面装的是数据，我们可以看到这里一共有两条数据，一条是newTodo的数据，类型是字符串，另一条是todos数据，类型是数组对象，然后我们现在再返回html中，查看那里用到了这两条数据的，可以看到<strong>v-model="newTodo"</strong>，所以我们暂时可以知道具有v-model属性的这个input输入框和newTodo这条数据绑定了起来，至于是单向绑定还是双向绑定还是仅仅是初始化的时候绑定一次，我们现在还不知道，接着看下面的todos，<strong>v-for="todo in todos"</strong>，可以明白的知道，这是个for遍历，类似于for(i in object)一样，而这里的意思就类似于在todos这个数组中遍历一下，然后在li标签内对遍历到的数据进行操作，下面就用到了字符串模版，将需要的数据提取出来就好了。
继续往下面看：

```
methods: {
    addTodo: function () {
      var text = this.newTodo.trim()
      if (text) {
        this.todos.push({ text: text })
        this.newTodo = ''
      }
    },
    removeTodo: function (index) {
      this.todos.splice(index, 1)
    }
}
```

这部分是methods，按照字面意思来看，叫做方法，也就是js中的函数（function），可以看到里面一共包含了两个function，一个是addTodo，另一个是removeTodo，回顾html文件，可以知道调用这两个函数方法的地方分别是<strong>v-on:keyup.enter</strong>和<strong>v-on:click</strong>，前一个就是类似于原生的onKeyup方法，而第二个就是onClick了，不过keyup后面跟了enter，说明这个keyup方法是和enter健绑定的，现在我们可以进入这两个函数内部看看，看它具体的实现方法。

```
addTodo: function () {
      var text = this.newTodo.trim()
      if (text) {
        this.todos.push({ text: text })
        this.newTodo = ''
      }
}
```

这个函数中主要是在针对this.newTodo进行操作，我们不难发现，newTodo这条数据是和v-model这个属性绑在一起的，然后这个属性又是和input输入框绑定在一起的，所以可以表明这里的this.newTodo指的就是input输入框里的值，然后再将其去除首尾空格，push进todos列表，最后再清空输入框。
而我们再来看下另一个函数：

```
removeTodo: function (index) {
      this.todos.splice(index, 1)
}
```

这个函数更简单，就一句话，执行一个数组操作，删除数组的某一项，而这个函数的调用在这里：v-on:click="removeTodo($index)"，可以看到在调用的时候传入了一个参数$index，这指的是对应的li标签的index，然后当点击的时候，就可以通过index来删除这个li标签了。

说了这么多，我知道，理工科的东西讲再多的道理都是十分抽象的，所以最后我还是上几张图吧，为了能够更好的理解。

![](http://oatasl78l.bkt.clouddn.com/vue%201.png)
图1
这是初始化的时候，一切都为空。

![](http://oatasl78l.bkt.clouddn.com/vue%202.png)
图2
然后我在输入框里输入数据，按下enter键。

![](http://oatasl78l.bkt.clouddn.com/vue%203.png)
图3
可以看到，下面列表中多了一条数据，是我刚才输入的数据。

![](http://oatasl78l.bkt.clouddn.com/vue%204.png)
图4
然后我又输入了几条数据，一共三条。

![](http://oatasl78l.bkt.clouddn.com/vue%205.png)
图5
最后我删除其中一条，就删除第二条吧，点击右边的那个小叉就可以了，可以看到这条就被删了，其他的数据不变。

















































