---
title: 页面表单后台配置化方案
---

### 写在前面

又是好久没有写博客了，之前写了太多纯技术相关的文章，后来吧，发现技术是技术，可是学习技术的最终点，是要在业务中实现落地，会的技能再多，不能在具体的业务场景下得到结果得到产出，都是不是好的技术，毕竟，公司也好社会也罢，都是结果导向的。

<!-- more -->

### 页面表单配置化方案

#### 开始

这个解决方案要从我所在的业务场景说起，我主要做的是crm和商家相关的，大多数都是对内的一些后台系统，其业务特点是：轻样式（基本不会用到什么绚丽的，高难度的效果），重交互（各种逻辑交叉纵横，剪不断，理还乱）。而又会涉及到各个系统之间的交流，比如面向买家的，面向商家的，面向客服的，这之间都会有联系，一个纠纷单（申诉，维权或举报），首先从买家端发起，经过处理，然后流到商家端，商家也会处理（或不处理，等到时间终止），最后流向客服端，客服来进行最终的判定，一个维权单才会算作结束。且买卖和商家的系统分别会涉及到PC和H5。

上面是crm的基本业务介绍，然后就开始进入正题，表单配置化。

这种To B的业务，最最最常用的，就是各种表单，而且逻辑也十分复杂；且由于涉及到的表单比较多，经常会涉及到表单的变动。最基本的方法就是前端在页面上添加一个表单组件，然后后端在接口增加那个组件对应的字段，这样就ok了，但是这会涉及一些问题或麻烦，比如每次只增加一个简单的输入框，却需要前端后端一起来工作，维护成本比较高；其次是组件化（模块）程度不够高，复用基本上靠的是复制加粘贴。

所以为了解决crm表单维护的麻烦，提高维护的效率，所以才作出了这套表单后台配置化的方案，将表单维护的前后端开发打通，以后要想再更改表单什么的，能够一个人（不管是不是技术人员）快速完成。

#### 我所接触的表单

最开始，是在PHP的老系统里，当时还是用的jQuery，当时处理表单变化的方案就是将表单分解为各个模版，比如一个select，下面有5个选项，那么你就需要5个模版，当使用者选择某一个选项时，展示对应的模版，如果有修改，就需要前端修改模版，后端修改接口，这也就是最基本的方法。这个我就不另外截图了。

下面要说的是某次开发中遇到的情况，这是一个表单，但是并不完整，可以看到，顶部的下拉框还属于未选择状态，然后当你选择下拉框的时候，会进行一个异步请求，将会收到这个下拉选项所对应的一些额外的表单数据，最后你用拿到的数据渲染出一些新的表单，再和原来已经存在的表单合并，就变成了一个新的表单。

这算是页面表单配置化的雏形吧，这个时候有了一点基本的表单后台配置化思想，但是问题还是有很多，比如组件不够全，渲染全靠遍历，没有涉及表单间的嵌套，表单的校验，表单事件是写死的，后台数据是后台开发人员配的，对其他人员使用不友好等等问题，但是已经迈出了第一步，那就是表单数据由接口获取。所以这个时候就有了个想法，既然部分表单组件可以是由接口动态获取数据渲染的，那么为什么不能将所有的表单组件都这样做呢？

最后，我们就开始了将这个想法具体实现，第一次就是在crm业务中的纠纷处理平台中，通过其特有的买家、卖家、客服，三端都有的关联，业务场景和实现环境都比较适合，所以才决定用在纠纷处理平台中。


#### 新版纠纷平台中的表单后台配置化

既然决定在纠纷处理平台中实践了，但是问题还是有很多的，比如之前的表单数据是后台直接写在代码中的，我们为了让非技术人员，主要是对应的客服人员，也能够友好的使用这个系统，我们专门在客服系统里做了一个可视化的表单配置界面（将配置页放在客服系统是因为这是内部系统，安全系数比较高，且又和商家、用户的系统有一定的关联，客服系统本身就是整个大项目结构中的一环，所以放在客服系统里），解决了非技术人员的使用问题，这是第一个问题；接下来是组件的丰富程度，这里组件就分为基本组件和业务组件，基本组件倒是比较简单，但是业务组件就比较麻烦了，千奇百怪的各种组件，且实用频率又不高，但是想到可能会给后面他人的维护带来很大的帮助，所以我们开始做的第一步，就是完善组件库，包括传图组件，原图组件，商品验证链接组件等等等等，这些具体的业务组件，我会在后面再做整理；组件够完善了，后台也都配置好了，接下来的问题就是前端渲染了，渲染中最大的一个问题就是数据多了，怎么高效地渲染？由于表单会有嵌套，且很可能是多层嵌套的存在，所以我们需要在渲染上进行一下优化，最基本的map循环遍历，然后渲染，后面如果每次发生变化，再map，这是最简单最基本的方法，但是由于效率太低，所以需要进行优化，我们最终选择的优化方法是拿到这个巨大的json对象后，第一次遍历一下，将多维的表单降维成一维的，并且在每个表单组件上都加上了parent和children属性，以及display属性，parent和children属性是用来找到当前表单所依赖的父组件和当前表单能够影响到的子组件，而最后的display就是表示当前表单的展示状态，这种遍历方法优化了很多的时间，可是没有数据就没有说服力，所以我专门通过模拟数据来获取它们的所耗时长，一组是优化前的数据，层级暂定于三层，每层1000个（数据太小了结果可信度不高，且耗时精度也不够高，虽然现实生活中基本不会有一个表单的表单组件达到3000个），另一种是3000个表单，层级仅仅是一级，最重模拟的结果是，第一种方法耗时299毫秒，而第二种方法耗时2毫秒，两种方法耗时差距不言而喻。

确定了基本思路后，然后就开始完善组件库，因为这种配置化的，一定需要有一个完善的组件库作为基础来支撑，能够让配置的时候来随意调用组件，毕竟巧妇难为无米之炊，这里的组件分为两类，一种是基础组件，在react中，几本的表单组件，通过一个Filed组件就几本能够搞定了，况且我们也有不止一个的react组件库（现在正在进行技术收敛中），所以在基础组件这方面，几本不需要担心什么，除了基础组件，然后就是千奇百怪的业务组件了吧，我稍微整理了一下，业务组件现在有这么几个：可展开与折叠的卡片布局组件，上传图片，带搜索功能的下拉框（里面又细分为单选、多选、同步、异步），图片库图片上传组件，地址组件，pc图片预览组件，h5图片预览组件，商品校验组件，带选中和排序的table，windows下自动调用打印和打印预览的组件，订单卡片组件，以及其他。。。

完善业务组件库是一项工作量不小的活，虽然我在这里就列了各个组件的名字，但是实际开发中还是需要克服各种各样的问题的；当组件库完善之后，现在就要正式开始项目的开发了，先在后台有一个表单配置的列表，你可以知道某人在某个时间修改了哪个表单，这样做的目的是出了问题能够更快速的找到相关责任人，整个后台的列表页，里面最主要的功能就是元件配置页，它有添加元件，元件排序，以及预览功能。
具体后台配置的页面，请参考下面图片。

![](http://oatasl78l.bkt.clouddn.com/complaint%E5%88%9D%E5%A7%8B%E5%8C%96.jpg)

这是后台界面，开始添加配置表单时候最开始的样子，这个页面现在还是空白的因为这个表单现在还没有添加任何的表单元件，右上角有三个按钮，分别是添加元件，保存元件排序，以及预览功能，保存元件排序功能和预览功能是一定要有表单元件后，才能执行操作的，所以我们先点击添加元件按钮。

![](http://oatasl78l.bkt.clouddn.com/complaint%E5%85%AC%E5%85%B1%E5%BF%85%E5%A1%AB%E6%9D%A1%E4%BB%B6.jpg)

![](http://oatasl78l.bkt.clouddn.com/complaint%E5%85%AC%E5%85%B1%E5%BF%85%E5%A1%AB.jpg)

上面两张图片，是点击添加元件后的弹窗，上面的中英文字段，长描述短描述，最底部的是否必填选项，还有默认值，都是组件通用的，每个组件都有这几个选项，唯一不通的是默认值会根据组件的不同类型，需要的默认值类型也不同，现在来说最重要的两个选项，那就是出现条件和类型，类型指的是你所需要的组件类型，它具体的类型如下图，由于弹窗下拉框中截图不方便，所以我直接从代码里截图吧，

![](http://oatasl78l.bkt.clouddn.com/complaint%E6%80%BB%E7%B1%BB%E5%9E%8B.jpg)

可以看到，它一共有这么多的组件类型，除了前面的五种选项是基本组件意外，其他的组件全是需要自己封装的业务组件，光封装这些组件就费了不小的力，至于出现条件那一栏，我们稍后再说，然后再看具体的各个类型的差异。

![](http://oatasl78l.bkt.clouddn.com/complaint%E7%9F%AD%E8%BE%93%E5%85%A5%E6%A1%86.jpg)

这是选择短文本框的情形，可以看到，选择了类型后，下面的表单会新出一个校验规则这一栏，里面就是根据当前选中的表单元件的类型，再来配置对应的校验数据，这里是文本框，校验规则可以是简单的字符长度限制，也可以直接传过来一段正则公式来进行匹配，当然，后面的长文本框也是类似，我就不再赘述了。

![](http://oatasl78l.bkt.clouddn.com/complaint%E5%8D%95%E9%80%89%E6%A1%86.jpg)

然后再来看单选框，多选框，下拉单选框，这几个也是一类的，他们所需要的数据都是key-value的键值对，这个加号，就会出现名-值对让你填写，这个名就是key，值就是value，展示的是名操作的是值，然后右边的上下箭头是选择数据的排序顺序，旁边的减号是删除这一行的数据。

![](http://oatasl78l.bkt.clouddn.com/complaint%E6%97%B6%E9%97%B4%E7%BB%84%E4%BB%B6%202.jpg)

然后就是日期组件，日期组件在后台配置的时候就比较简单，主要问题是在前端如何能够将日期组件和对应的change事件良好的绑定起来，日期组件只需要传一个默认的时间日期就好了，也可以不传默认值。

![](http://oatasl78l.bkt.clouddn.com/complaint%E5%9B%BE%E7%89%87%E4%B8%8A%E4%BC%A0.jpg)

接下来是图片上传组件，最开始设想的时候，想着所有校验规则都由前端来做，所以有了图片大小的校验规则，后来发现前端做这个校验比较麻烦，所以这个由后端来做，前端的这个配置功能也暂时没有下掉，至于图片数量和格式，就比较容易校验了，而这个图片上传组件，在开发的时候，也考虑了要不要缩略图，后来决定不将缩略图包含在内，因为并不是所有图片上传都需要缩略图，而又由于缩略的排版样式以及删除，都不一样，所以我们将这个功能放在组件外部，让需要的人自己去实现缩略功能，uni图片上传和原图上传，在后端配置来说，也是类似（但是在前端展示可就千差万别了）。

![](http://oatasl78l.bkt.clouddn.com/%E5%95%86%E5%93%81%E9%93%BE%E6%8E%A5%20%E9%A9%AC%E8%B5%9B%E5%85%8B.png)

这是商品链接的组件，首先是一个输入框，旁边有一个校验按钮，你往这个输入框里填入一段商品链接，然后点击校验，会向后端发请求，分为自己的商品，xxx网站商品，aaa网站商品等，如果成功，则会展示商品的链接和缩略图展示，如果失败，则是一段错误提示的语句。校验规则比较简单，就是需不需要限制为xxx链接。还有就是数量限制。

后面的注册时间注册地址，就是比较简单的了，也没有什么好说的，下面我主要说一下之前留下的问题，出现条件这一栏吧。

最开始知道要做层级的时候，是十分抗拒的，因为特别麻烦，用不断的去遍历来解决，也特别low，特别没效率；但是当我仔细分析后我发现，实际上层级嵌套，有两个关键的节点，就是单选框和下拉框（这两个本质都是一样的，都是多个选项选一个），所以只要抓住这两个关键的节点，层级的嵌套，就十分简单了，所以当没有单选框和下拉框的时候，就肯定不会存在表单的层级嵌套，所以出现条件就是空，也就是表示默认的第一层，最外面的层级，当有了下拉框或者单选框的时候，就会变成下面这种：

![](http://oatasl78l.bkt.clouddn.com/complaint%E5%B1%82%E7%BA%A7%20%E5%87%BA%E7%8E%B0%E6%9D%A1%E4%BB%B6.jpg)

你可以选择在哪一个（下拉框或单选框）组件时（出现条件的第一个下拉框），选中什么值的时候（出现条件的第二个下拉框），才出现你当前配置的这个表单元件，然后不断重复，就可以完成你想要的层级关系。

然后在表单外面也是可以有排序，增删功能的，这里要额外说一下这个预览功能，第一版本是没有的，所以配置好了要立即看到结果还比较麻烦，现在有了预览，配置好了下一秒就可以看到结果，大大的提高了效率。

后台配置好了，接下来是前端的渲染，我们将后台配置的所有数据作为一个十分庞大的json对象，直接返回给前端展示页来展示，前面已经说了，前台页面拿到这些数据后，会将多层级的数据拍平改为一级的数据，并且为了不保持其原有的层级信息，将每一个组件里都增加了parent和children属性，来表达其所依赖的父组件以及下面影响的子组件，然后自己通过switch方法，将所有可能出现的组件类型做成一个渲染的模版，包括将这个组件内的所有的信息都绑定在对应组件的属性上，出了上面说了parent，children，display等，还有checkRule，name，操作函数func，内置的数据等等，而且据我们所观察，表单层级嵌套，只有在下拉框和大选框的时候才会存在（其实下拉框和单选框本质上是一个意思，都是多选一的意思），所以我们会在下拉框和单选框的时候对数据进行遍历，这样就可以免去很多无用的遍历，大大提高效率。且最终我们提交的时候，会将拿到的所有数据，分别与其组件中的checkRule相进行校验，如果全部通过，才能进行提交，而这个校验规则checkRule，也是根据不同的组件变化的，比如说图片上传组件中最大最小张的限制，多选按钮中至少选一项等等，而普通的输入框就比较简单了，后台直接传过来一个正则公式相匹配就好了。

所以总的来说，后台的主要难点是层级嵌套在可视化界面上的配置，并且为了将配置的结果做到所见即所得，所以有一个预览功能，能够配置好了，下一秒就能看到结果，前台方面就是高效地渲染，以及各个组件提交前的校验，这样配置出来的表单，就做到了一劳永逸，除非后面有十分大的业务变更，否则后续维护的人力成本可以说是很少很少了。

总结：所以总的来说，这就是一个针对系统页面上表单比较多的时候，制订的一个表单后台配置化的方案，使用者（不管是不是技术人员）通过后台的可视化配置界面，能够即时性地对页面上的表单模版进行增删改，嵌套与校验等操作，最大的优点就是快捷迅速简单，不再像以前一样，前端改页面，后端改接口字段，然后测试，ok之后最后发布。













