---
title: fastClick 基本解读
---

### fast click诞生背景与使用

做过移动端前端的同学，应该都知道“点击延迟”，意思就是原生js的click事件在移动浏览器上会有300毫秒的延迟，这样就会让用户觉得卡顿，影响用户体验。为什么会有延迟呢？因为移动浏览器都是支持双击的，比如双击缩放或双击滚动，所以当用户第一次点击屏幕后，浏览器并不知道你是单击还是双击，因此所有的浏览器都效仿了Safari的做法，那就是等待300毫秒。

现在移动端大行其道，已经占据了主流。最显著的一个例子就是，天猫双十一的时候，超过百分之八十的订单都来自于移动端，这也就说明了移动端现在的火热程度。在这个前提下，你移动端的每个点击事件都有300毫秒的延迟，那是绝对不允许的。

后面也出现了很多的解决办法，比如Zepto的tab事件(会引发击穿tab的bug)，fastClick.js等等，所以现在我就准备来好好讲解一下fastClick。

<!-- more -->

### 解析

##### 引入fastClick

源码的第829行：选择引入fastClick的方式，具体代码如下：

```
	if (typeof define === 'function' && typeof define.amd === 'object' && define.amd) {

		// 优先兼容AMD的方式
		define(function() {
			return FastClick;
		});
	} else if (typeof module !== 'undefined' && module.exports) {
		// 其次是commonJs
		module.exports = FastClick.attach;
		module.exports.FastClick = FastClick;
	} else {
		// 最后兼容原生js
		window.FastClick = FastClick;
	}
```

##### 入口

源码第824行：fastClick入口方法 attach

```
// 这里是通过return一个newFastClick函数来进行继承的
// 其中layer参数是指的要监听的dom对象
// option是用来覆盖自定义参数
FastClick.attach = function(layer, options) {
	return new FastClick(layer, options);
};
```
##### FastClick 主函数

第23行到103行，这里主要是设置一些参数的默认值，这些参数就是FastClick的精华所在，也就是上面option参数中的默认参数值，其代码大致如下。

```
this.trackingClick = false;
this.trackingClickStart = 0;
this.targetElement = null;
this.touchStartX = 0;
this.touchStartY = 0;
this.lastTouchIdentifier = 0;
this.touchBoundary = options.touchBoundary || 10;
this.layer = layer;
this.tapDelay = options.tapDelay || 200;
this.tapTimeout = options.tapTimeout || 700;
```

##### 判断是否需要fastClick

第105到107行，判断是否需要引入fastClick，如果不需要则return。

```
if (FastClick.notNeeded(layer)) {
	return;
}
```

话说回来，并不是任何时候都需要引入fastClick的，其官网上面就说得很清楚，下面几种情况，是不需要引入fastClick的。

1，所有pc浏览器。

2，浏览器不支持ontouchstart。

3，安卓中chrome meta中有禁止缩放（user-scalable="no"）。

4，安卓中chrome meta中有width="device-width"。

5，黑莓 10.3+。

6，firefox 27+。

7，有-ms-touch-action: manipulation属性的IE10。

8，有touch-action: manipulation属性的IE11。

##### 将自定义的函数绑定在传入的dom节点上

第110行到132行。

```
layer.addEventListener('touchstart', this.onTouchStart, false);
layer.addEventListener('touchmove', this.onTouchMove, false);
layer.addEventListener('touchend', this.onTouchEnd, false);
layer.addEventListener('touchcancel', this.onTouchCancel, false);
```

##### 对旧版本android不支持 stopImmediatePropagation 事件的兼容

第137行到159行。

这里可以简单说一下 stopImmediatePropagation 和  stopPropagation 的区别吧。stopPropagation是阻止事件冒泡到父级元素，这是最基本的，而stopImmediatePropagation除了能够达到这个要求外，还额外执行了一件事，就是阻止绑定在事件元素的其它同类事件的回调函数的运行，比如下面的示例：

```
$("p").click(function(event) {
  event.stopImmediatePropagation();
});
$("p").click(function(event) {
  // 不会执行以下代码
  $(this).css("background-color", "#f00");
});
```

##### 兼容直接绑定在dom元素上的onclick事件

第164行到173行。

```
if (typeof layer.onclick === 'function') {
  oldOnClick = layer.onclick;
  layer.addEventListener('click', function(event) {
    oldOnClick(event);
  }, false);
  layer.onclick = null;
}
```

先判定html元素的元素内部是否直接写了onclick事件，类似于这样：<div onclick="func()"></div>，如果有的话，将其使用js绑定在dom上，然后将html元素内的事件清空。

这是为了之后的模拟点击事件。

##### 判断用户终端

178-221

通过navigator.userAgent来判断用户的终端类型，其中有个问题是“Windows Phone 8.1 fakes user agent string to look like Android and iPhone.”，所以在判断安卓和iOS的时候需要有一个判断前提，就是非window phone的时候。

##### needsClick函数，needsFocus函数

227-254

判断哪些元素需要原生的click事件，哪些需要原生的focus事件。具体代码如下：

```
FastClick.prototype.needsClick = function(target) {
	switch (target.nodeName.toLowerCase()) {

	// Don't send a synthetic click to disabled inputs (issue #62)
	case 'button':
	case 'select':
	case 'textarea':
		if (target.disabled) {
			return true;
		}

		break;
	case 'input':
		// File inputs need real clicks on iOS 6 due to a browser bug (issue #68)
		if ((deviceIsIOS && target.type === 'file') || target.disabled) {
			return true;
		}

		break;
	case 'label':
	case 'iframe': // iOS8 homescreen apps can prevent events bubbling into frames
	case 'video':
		return true;
	}

	return (/\bneedsclick\b/).test(target.className);
};

FastClick.prototype.needsFocus = function(target) {
	switch (target.nodeName.toLowerCase()) {
	case 'textarea':
		return true;
	case 'select':
		return !deviceIsAndroid;
	case 'input':
		switch (target.type) {
		case 'button':
		case 'checkbox':
		case 'file':
		case 'image':
		case 'radio':
		case 'submit':
			return false;
		}

		// No point in attempting to focus disabled inputs
		return !target.disabled && !target.readOnly;
	default:
		return (/\bneedsfocus\b/).test(target.className);
	}
};
```

上面这些就是哪些要原生click，原生focus的元素。如果需要，只需要在dom元素的class中加上“needsClick”或者“needsFocus”。像这样：

```
<div class="needsclick">aaaa</div>
```

##### 各种判断

311-319行：determineEventType 兼容安卓chrome中的select框事件从click改为mousedown

```
FastClick.prototype.determineEventType = function(targetElement) {
	//Issue #159: Android Chrome Select Box does not open with a synthetic click event
	if (deviceIsAndroid && targetElement.tagName.toLowerCase() === 'select') {
		return 'mousedown';
	}

	return 'click';
};
```

327-337行：focus函数 兼容苹果手机(iOS7)setSelectionRange不能正确获取焦点的bug

```
FastClick.prototype.focus = function(targetElement) {
	var length;

	// Issue #160: on iOS 7, some input elements (e.g. date datetime month) throw a vague TypeError on setSelectionRange. These elements don't have an integer value for the selectionStart and selectionEnd properties, but unfortunately that can't be used for detection because accessing the properties also throws a TypeError. Just check the type instead. Filed as Apple bug #15122724.
	
	if (deviceIsIOS && targetElement.setSelectionRange && targetElement.type.indexOf('date') !== 0 && targetElement.type !== 'time' && targetElement.type !== 'month') {
		length = targetElement.value.length;
		targetElement.setSelectionRange(length, length);
	} else {
		targetElement.focus();
	}
};
```

376-384行：getTargetElementFromEventTarget函数，兼容获取点击元素，iOS 4.1中会获取文字作为焦点，取它的父元素dom

```
FastClick.prototype.getTargetElementFromEventTarget = function(eventTarget) {
	// On some older browsers (notably Safari on iOS 4.1 - see issue #56) the event target may be a text node.
	if (eventTarget.nodeType === Node.TEXT_NODE) {
		return eventTarget.parentNode;
	}
	return eventTarget;
};
```

497-512行：findControl函数，为了点击label元素的时候能改找到其所对应的表单元素的焦点

```
FastClick.prototype.findControl = function(labelElement) {
	// Fast path for newer browsers supporting the HTML5 control attribute
	if (labelElement.control !== undefined) {
		return labelElement.control;
	}

	// All browsers under test that support touch events also support the HTML5 htmlFor attribute
	if (labelElement.htmlFor) {
		return document.getElementById(labelElement.htmlFor);
	}

	// If no for attribute exists, attempt to retrieve the first labellable descendant element
	// the list of which is defined here: http://www.w3.org/TR/html5/forms.html#category-label
	return labelElement.querySelector('button, input:not([type=hidden]), keygen, meter, output, progress, select, textarea');
};
```

461-469：touchHasMoved 手指点击时移动间距大于10px，返回true

```
FastClick.prototype.touchHasMoved = function(event) {
	var touch = event.changedTouches[0], boundary = this.touchBoundary;

	if (Math.abs(touch.pageX - this.touchStartX) > boundary || Math.abs(touch.pageY - this.touchStartY) > boundary) {
		return true;
	}

	return false;
};
```

478-490：onTouchMove 手指点击时移动间距大于10px，即视为touchmove，不触发模拟click事件

```
FastClick.prototype.onTouchMove = function(event) {
	if (!this.trackingClick) {
		return true;
	}

	// If the touch has moved, cancel the click tracking
	if (this.targetElement !== this.getTargetElementFromEventTarget(event.target) || this.touchHasMoved(event)) {
		this.trackingClick = false;
		this.targetElement = null;
	}

	return true;
};
```

#### 核心方法

393-452：onTouchStart函数，tapDelay默认300毫秒，点击时间差小于300毫秒，则阻止事件再次触发，阻止短时间内双击的问题

```
FastClick.prototype.onTouchStart = function(event) {
	var targetElement, touch, selection;
	if (event.targetTouches.length > 1) {
		return true;
	}

	targetElement = this.getTargetElementFromEventTarget(event.target);
	touch = event.targetTouches[0];

	if (deviceIsIOS) {
		selection = window.getSelection();
		if (selection.rangeCount && !selection.isCollapsed) {
			return true;
		}

		if (!deviceIsIOS4) {
			if (touch.identifier && touch.identifier === this.lastTouchIdentifier) {
				event.preventDefault();
				return false;
			}

			this.lastTouchIdentifier = touch.identifier;
			this.updateScrollParent(targetElement);
		}
	}

	this.trackingClick = true;
	this.trackingClickStart = event.timeStamp;
	this.targetElement = targetElement;

	this.touchStartX = touch.pageX;
	this.touchStartY = touch.pageY;

	if ((event.timeStamp - this.lastClickTime) < this.tapDelay) {
		event.preventDefault();
	}

	return true;
};
```

523-612：onTouchEnd函数

```
FastClick.prototype.onTouchEnd = function(event) {
	var forElement, trackingClickStart, targetTagName, scrollParent, touch, targetElement = this.targetElement;

	if (!this.trackingClick) {
		return true;
	}

	if ((event.timeStamp - this.lastClickTime) < this.tapDelay) {
		this.cancelNextClick = true;
		return true;
	}

	if ((event.timeStamp - this.trackingClickStart) > this.tapTimeout) {
		return true;
	}

	this.cancelNextClick = false;

	this.lastClickTime = event.timeStamp;

	trackingClickStart = this.trackingClickStart;
	this.trackingClick = false;
	this.trackingClickStart = 0;

	if (deviceIsIOSWithBadTarget) {
		touch = event.changedTouches[0];

		targetElement = document.elementFromPoint(touch.pageX - window.pageXOffset, touch.pageY - window.pageYOffset) || targetElement;
		targetElement.fastClickScrollParent = this.targetElement.fastClickScrollParent;
	}

	targetTagName = targetElement.tagName.toLowerCase();
	if (targetTagName === 'label') {
		forElement = this.findControl(targetElement);
		if (forElement) {
			this.focus(targetElement);
			if (deviceIsAndroid) {
				return false;
			}

			targetElement = forElement;
		}
	} else if (this.needsFocus(targetElement)) {
		if ((event.timeStamp - trackingClickStart) > 100 || (deviceIsIOS && window.top !== window && targetTagName === 'input')) {
			this.targetElement = null;
			return false;
		}

		this.focus(targetElement);
		this.sendClick(targetElement, event);

		// Also this breaks opening selects when VoiceOver is active on iOS6, iOS7 (and possibly others)
		if (!deviceIsIOS || targetTagName !== 'select') {
			this.targetElement = null;
			event.preventDefault();
		}

		return false;
	}

	if (deviceIsIOS && !deviceIsIOS4) {

		if (scrollParent && scrollParent.fastClickLastScrollTop !== scrollParent.scrollTop) {
			return true;
		}
	}

	if (!this.needsClick(targetElement)) {
		// 如果这不是一个需要使用原生click的元素，则屏蔽原生事件，避免触发两次click
		// 触发一次模拟的click
		event.preventDefault();
		this.sendClick(targetElement, event);
	}

	return false;
};
```

##### 296-311 sendClick（核心方法）

```
FastClick.prototype.sendClick = function(targetElement, event) {
	var clickEvent, touch;

	if (document.activeElement && document.activeElement !== targetElement) {
		document.activeElement.blur();
	}

	touch = event.changedTouches[0];

	// 合并成一个点击事件，通过一个额外的属性，确保事件能够被跟踪
	// 最后再通过模拟一个点击事件来执行，事件触发器dispatchEvent参考这篇文章 http://blog.csdn.net/magic__man/article/details/51831227
	clickEvent = document.createEvent('MouseEvents');
	clickEvent.initMouseEvent(this.determineEventType(targetElement), true, true, window, 1, touch.screenX, touch.screenY, touch.clientX, touch.clientY, false, false, false, false, 0, null);
	clickEvent.forwardedTouchEvent = true;
	targetElement.dispatchEvent(clickEvent);
};
```

### Zepto 点击穿透与 FastClick

移动端点击事件中，Zepto中也解决了移动浏览器中300毫秒的事情，但是就出现了各种击穿现象。

1，同页面tap点击弹出弹层，弹层中也有一个button，正好重叠的时候，会出现击穿。

2，tap事件点击，页面跳转，新页面中同位置也有一个按钮，会出现击穿。

#### 解释击穿

zepto中的 tap 通过兼听绑定在 document 上的 touch 事件来完成 tap 事件的模拟的，是通过事件冒泡实现的。在点击完成时（touchstart / touchend）的 tap 事件需要冒泡到 document 上才会触发。而在冒泡到 document 之前，手指接触和离开屏幕（touchstart / touchend）是会触发 click 事件的。

因为 click 事件有延迟（大概是300ms，为了实现safari的双击事件的设计），所以在执行完 tap 事件之后，弹出层立马就隐藏了，此时 click 事件还在延迟的 300ms 之中。当 300ms 到来的时候，click 到的其实是隐藏元素下方的元素。

如果正下方的元素有绑定 click 事件，此时便会触发，如果没有绑定 click 事件的话就当没发生。如果正下方的是 input 输入框（或是 select / radio / checkbox），点击默认 focus 而弹出输入键盘，也就出现了上面的“点透”现象。

### 击穿的解决办法

#### 遮挡

由于移动端点击click事件的延后性，导致原本想点击的元素消失了，直接点击到了下面的元素，于是便有了穿透。于是最简单的办法就是在你点击的下方增加一层遮挡的元素，让其300毫秒后自动消失就行了；或者不用延时动画，改为增加一个透明的元素来隔离，也是可以的；这样就从dom方面对其进行了隔离，解决了穿透的问题。

#### pointer-events

pointer-events是CSS3中的属性，它有很多取值，有用的主要是auto和none，其他属性值为SVG服务。

如果pointer-events的属性值为none，元素不再是鼠标事件的目标，鼠标不再监听当前层而去监听下面的层中的元素。但是如果它的子元素设置了pointer-events为其它值，比如auto，鼠标还是会监听这个子元素的。具体实现代码如下：

```
$('#closePopup').on('tap', function(e){
    $('#popupLayer').hide();
    $('#bgMask').hide();

    $('#underLayer').css('pointer-events', 'none');

    setTimeout(function(){
        $('#underLayer').css('pointer-events', 'auto');
    }, 400);
});
```

#### fastClick

使用fastclick库，其实现思路是，取消 click 事件，用 touchend 模拟快速点击行为。只需要一句```FastClick.attach(document.body);```

从此所有的点击事件，都是用click且不会出现穿透问题，并且没有300毫秒的延迟。

下一篇博客，我会对js的事件机制做一个详细的分析，包括移动端的。






















































































