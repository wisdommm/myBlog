---
title: vue.js 组件快速入门（一）
---

前几天读了一篇好文章，[vue组件化](http://www.cnblogs.com/keepfool/p/5625583.html)，这篇算是我自己提炼出来的的归纳与总结吧。

在前端开发中，以前我们会把常用的公共的方法（函数）提取出来，整合为一个库，最有名的当属jQ了吧，然后到了react出现，这些类库主要都是针对view层进行的操作与封装，我们又可以把view层面上的一些东西抽出来进行封装和整理，比如一个table，一个button，一个div等，也可以像曾经提取函数一样抽出来，写成公用的，在需要使用的地方直接引入就可以了，当然，也可以根据你实际运用的需要进行适当的自定义配置，而这些被抽出来共用的table，button，div这些，就叫做组件。

上面是我自己瞎扯的，原文作者讲得比较繁琐，所以我就用自己的语言来概述了一下，虽然这篇文章是我转载的，不过我也参考了其它的文章／博客，知识点的相似度接近80%，但是语言基本上都是我自己重新组织的，所以加了很多自己的想法和思考，下面我们开始进入正题。

<!-- more -->

###vue组件，创建与注册的基本步骤。

vue组件的使用，有三个步骤，分别是创建组件，注册组件和使用组件，如下图所示。
![](http://oatasl78l.bkt.clouddn.com/vue1.png)

我们可以用一段代码来演示这三个步骤：

```
<!DOCTYPE html>
<html>
    <body>
        <div id="app">
            <!-- 3. #app是Vue实例挂载的元素，应该在挂载元素范围内使用组件-->
            <my-component></my-component>
        </div>
    </body>
    <script src="js/vue.js"></script>
    <script>
    
        // 1.创建一个组件构造器
        var myComponent = Vue.extend({
            template: '<div>This is my first component!</div>'
        })
        
        // 2.注册组件，并指定组件的标签，组件的HTML标签为<my-component>
        Vue.component('my-component', myComponent)
        
        new Vue({
            el: '#app'
        });
        
    </script>
</html>
```

运行结果很简单，就是一句“This is my first component!”，可以看到，使用组件和使用普通的HTML元素没什么区别，你可以就把组件当做一个HTML元素来使用。

要理解组件的创建和注册，我们可以用一下几个步骤来详细解释：

1. Vue.extend()是Vue构造器的扩展，调用Vue.extend()创建的是一个组件构造器，而不是一个具体的组件实例。
2. Vue.extend()构造器有一个选项对象，选项对象的template属性用于定义组件要渲染的HTML。 
3. 使用Vue.component()注册组件时，需要提供2个参数，第1个参数时组件的标签，第2个参数是组件构造器。 
4. Vue.component()方法内部会调用组件构造器，创建一个组件实例。 
5. 组件应该挂载到某个Vue实例下，否则它不会生效。

我们一句一句来分析，先是创建一个组件，代码是

```
myComponent = Vue.extend({
	template: '<div>This is my first component!</div>'
})
```

创建组件很简单，直接调用vue构造器的扩展方法就行了，里面会传入一个对象，对象里面有个template属性，它指代的是这个组件的view的内容，是不是像react里面的render函数啊，它也是return的view内容，<strong>它们有哪些不同呢？</strong>毕竟学习知识要针对各种不同的但是有类似的知识点来进行归纳，整理和综合，才能进行融会贯通的运用。毕竟古人云：学而时习之，不亦悦乎！

好了，这是创建，下面是注册了，注册也很简单，一句代码就搞定了：Vue.component('my-component', myComponent);，使用Vue.component方法，传入两个参数，第一个参数是你想注册的组件名，第二个参数是你创建的组件名，是不是有点混淆？先是你通过Vue.extend创建了一个组件，名字叫myComponent，然后为了在HTML中能够当做标签来使用，你需要给你创建的这个组件注册一个组件名，名字叫做my-component，所以你在HTML就可以直接把你注册的组件名当做HTML标签来使用，像这样<my-component></my-component>就可以了。<br>
现在组件创建好了，也注册好了，可是还不能直接使用，因为还需要new一个Vue实例，才能使用，像这样

```
Vue({
    el: '#app'
});
```

一个Vue实例接收一个对象，里面的el属性是你组件的挂载节点，于是我又情不自禁地联想到了react，ReactDOM.render( dom , root)，就像这样，react也是将一个view界面的虚拟dom挂载到一个root节点上，而这里的Vue同样，需要将刚才的组件也挂载到某个节点下，这两者之间的异同后面再说吧，先说一点主要的，vue里的节点指的是vue组件的作用域，这点是和react差异比较大的一点，你将vue组件挂载在了某个节点下，意思是这个组件只有在这个节点范围内才可以使用，至于是使用多次，使用一次，还是一次都不使用，都没问题。而我们这个实例上，就是挂载在了id为app的节点上，所以在这个节点的范围内都可以使用<my-component></my-component>的形式，来调用vue组件myComponent。同理，如果你想在多个节点的作用域内使用某个vue组件的话，那你就得多new几次，把组件的挂载到多个节点下面就好了，而如果在没有实例化的组件中非法调用某个vue组件，它将不起任何作用。
详细情况可以看以下代码：

```
<!DOCTYPE html>
<html>
    <body>
        <div id="app1">
            <my-component></my-component>
        </div>
        <div id="app2">
            <my-component></my-component>
        </div>
        <div id="app3">
	        <!--该组件不会被渲染-->
	        <my-component></my-component>
	    </div>
    </body>
    <script>
        var myComponent = Vue.extend({
            template: '<div>This is a component!</div>'
        })
        
        Vue.component('my-component', myComponent)
        
        var app1 = new Vue({
            el: '#app1'
        });
        
        var app2 = new Vue({
            el: '#app2'
        })
    </script>
</html>
```

在上面代码中，我们创建了一个名叫myComponent的vue组件，然后注册了一个名叫my-component的组件来当做HTML标签使用，最后我们实例化时，只在id="app1"和id="app2"中挂载了这个vue组件，而在实际使用的时候，我们在id="app3"中也使用了这个<my-component></my-component>组件，所以最后，只有id="app1"和id="app2"的div中组件成功地被替换成了“This is a component!”，而id="app3"中的<my-component></my-component>标签将不会被渲染。具体效果如下图所示：
![](http://oatasl78l.bkt.clouddn.com/vue2.png)

好了，一个基本的vue组件的一整套流程基本就完全是这样了，any problems else？我知道，现在还有很多的问题没有解决，比如创建组件，注册组件和实例化挂载到某个节点下，整体步骤比较繁琐；注册组件的时候可以局部注册吗；组件可以嵌套吗？如果可以，如何嵌套；vue组件怎么怎么做成可扩展的？又怎样变为自定义配置呢？

我下一篇博客再细细道来。