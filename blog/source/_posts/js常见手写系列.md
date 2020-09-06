---
title: js常见手写系列
---

```
// 一，filter

Array.prototype.newFilter = function(fn , context){
    if( !Array.isArray(this) ){
        throw Error('not array');
        return;
    }
    let res = [];
    let arr = this;
    for(let i = 0 ; i < arr.length ; i++){
        let tmp = fn.call(context , arr[i] , i);
        if(tmp){
            res.push(tmp);
        }
    }
    return res;
}
```

<!-- more -->

```
// 二，map

Array.prototype.map = function(fn , context){
    if( !Array.isArray(this) ){
        throw Error('not array');
        return;
    }
    let res = [];
    let arr = this;
    for(let i = 0 ; i < arr.length ; i++){
        let tmp = fn.call(context, arr[i] , i , arr);
        res.push(tmp);
    }
    return res;
}

// 三，call/apply

Function.prototype.newCall = function(obj , ...args){
    obj = obj || window;
    bj.fn = this;
    let res = obj.fn(...args);
    delete obj.fn;
    return res;
}

Function.prototype.newApply = function(obj , arr){
    obj = obj || window;
    bj.fn = this;
    let res = obj.fn(...arr);
    delete obj.fn;
    return res;
}

// 四，new

function newOption(fn,...args){
    if(typeof fn !== 'function'){
        throw Error('not function');
    }
    let obj = {};
    obj._proto_ = fn.prototype;
    let res = fn.apply(obj , args);
    return typeof res === 'object' ? res : obj;
}

// 五，bind

Function.prototype.newBind = function(context , ...args1){
    let self = this;
    let fNOP = function(){}
    let fBound = function(...args2){
        self.apply( this instanceof fBound ? this : context , args1.concat(args2) );
    }
    fNop.prototype = this.prototype;
    fBound.prototype = new fNop();
    return fBound;
}

// 六，防抖
// 简易版
function debounce(fn , wait){
    let timer = null;
    let context = this;
    return function(...args){
        clearTimeout(timer);
        setTimeout(()=>{
            fn.apply(context , args);
        },wait)
    }
}

// 完全版
function debounce(fn , wait , immediate){
    let timer;
    let result;
    let debounced = function(...args){
        let context = this;
        if(timer) clearTimeout(timer);
        if(immediate){
            let callNow = !timer;
            timer = setTimeout(()=>{
                timer = null;
            },wait)
            if(callNow)result = fn.apply(context , args);
        }else{
            timer = setTimeout(()=>{
                fn.apply(context , args);
            },wait);
        }
        return result;
    }
    debounced.cancel = function(){
        clearTimeout(timer);
        timer = null;
    }
}

// 七，节流

// 简易版
function throttle(fn , wait){
    let context = this;
    let prev = 0;
    let timer = null;
    return function(...args){
        clearTimeout(timer);
        let now = +new Date();
        if(now - prev > wait){
            prev = now;
            fn.apply(context , args);
        } else {
            timer = setTimeout(fn, time);
        }
    }
}

// 完全版
function throttle(func, wait, options) {
    var timeout, context, args, result;
    var previous = 0;
    if (!options) options = {};

    var later = function() {
        previous = options.leading === false ? 0 : new Date().getTime();
        timeout = null;
        func.apply(context, args);
        if (!timeout) context = args = null;
    };

    var throttled = function() {
        var now = new Date().getTime();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
            if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(later, remaining);
        }
    };
    throttled.cancel = function() {
        clearTimeout(timeout);
        previous = 0;
        timeout = null;
    };
    return throttled;
}

// 八，deepClone深拷贝

function deepClone(obj){
    if(typeod obj !== 'object')return;
    let newObj = obj instanceof Array ? [] : {};
    for(let i in obj){
        if( obj.hasOwnProperty(i) ){
            // shallowClone
            // newObj[key] = obj[key];

            // deepClone
            newObj[i] = typeof obj[i] === 'object' ? deepClone(obj[i]) : obj[i];
        }
    }
    return newObj;
}

// 九，数组flatten

function flatten(arr , n){
    if( !Array.isArray(arr) )return;
    let res = [];
    debugger
    if(n > 0){
        for(let i in arr){
            if(arr[i] instanceof Array){
                res = res.concat( flatten( arr[i] , n-1 ) );
            }else{
                res.push(arr[i]);
            }
        }
    }else{
        res = res.concat(arr);
    }
    return res;
}

// 十，实现一系列函数，能够达到如下调用：one(add(two())) 输出 3；two(add(one())) 输出 3。

function add(...args1){
    return function(...args2){
        return args1.concat(args2).reduce((a,b)=>a+b);
    }
}

function one(fn){
    return fn ? fn(1) : 1;
}

function two(fn){
    return fn ? fn(2) : 2;
}

// 十一，instanceof

function instanceof(left , right){
    left = left._proto_;
    let prototype = right.prototype;
    while(true){
        // 核心原理是，在实例对象上不断通过_proto_在原型链上查找，看是否能找到原型的prototype
        if(!left)return false;
        if(left === prototype)return true;
        left = left._proto_;
    }
}

// 十二，实现一个EventEmitter，支持on、off、once、emit事件

class EventEmitter{
    constructor(){
        this.list = {};
    }

    on(name , fn){
        if(!this.list[name])this.list[name] = [fn];
        this.list[name].push(fn);
    }

    off(name , fn){
        if( this.list[name] ){
            let index = this.list[name].indexOf(fn);
            index > -1 ? this.list[name].splice(index , 1) : null;
        }
    }

    once(name , fn){
        let self = this;
        this.on(name , one);
        function one(...args){
            fn.call(self , ...args)
            self.off(name, one);
        }
    }

    emit(name , ...args){
        if(this.list[name]){
            this.list[name].forEach(fn=>{
                fn.call(this , ...args);
            });
        }
    }
}

// 十三，数据双向绑定

//es5版，核心是对象描述符
    Object.defineProperty(obj, "value", {
        get: function () {
          return value;
        },
        set: function (newValue) {
          value = newValue;
          // 当数据进行改变，触发对应的响应事件
          // 。。。
        }
    });

// es6版，核心是proxy
let handler = {
    set(target , key , value){
        target[key] = value;
        // Reflect写法
        Reflect.set(target, key, value)；
    },
    get(target , key){
        return target[key];
        // Reflect写法
        return Reflect.get(target, key);
    }
}
let p = new Proxy({} , handler);
// p.value = 123
// p.a
```

十四，观察者模式

```
function ObserverList(){
  this.observerList = [];
}
ObserverList.prototype.add = function( obj ){
  return this.observerList.push( obj );
};
ObserverList.prototype.count = function(){
  return this.observerList.length;
};
ObserverList.prototype.get = function( index ){
  if( index > -1 && index < this.observerList.length ){
    return this.observerList[ index ];
  }
};
ObserverList.prototype.indexOf = function( obj, startIndex ){
  var i = startIndex;
  while( i < this.observerList.length ){
    if( this.observerList[i] === obj ){
      return i;
    }
    i++;
  }
  return -1;
};
ObserverList.prototype.removeAt = function( index ){
  this.observerList.splice( index, 1 );
};

//目标
function Subject(){
  this.observers = new ObserverList();
}
Subject.prototype.addObserver = function( observer ){
  this.observers.add( observer );
};
Subject.prototype.removeObserver = function( observer ){
  this.observers.removeAt( this.observers.indexOf( observer, 0 ) );
};
Subject.prototype.notify = function( context ){
  var observerCount = this.observers.count();
  for(var i=0; i < observerCount; i++){
    this.observers.get(i).update( context );
  }
};

//观察者
function Observer(){
  this.update = function(){
    // ...
  };
}
```

十五，发布/订阅模式

```
var pubsub = {};
(function(myObject) {
    // Storage for topics that can be broadcast
    // or listened to
    var topics = {};
    // An topic identifier
    var subUid = -1;
    // Publish or broadcast events of interest
    // with a specific topic name and arguments
    // such as the data to pass along
    myObject.publish = function( topic, args ) {
        if ( !topics[topic] ) {
            return false;
        }
        var subscribers = topics[topic],
            len = subscribers ? subscribers.length : 0;
        while (len--) {
            subscribers[len].func( topic, args );
        }
        return this;
    };
    // Subscribe to events of interest
    // with a specific topic name and a
    // callback function, to be executed
    // when the topic/event is observed
    myObject.subscribe = function( topic, func ) {
        if (!topics[topic]) {
            topics[topic] = [];
        }
        var token = ( ++subUid ).toString();
        topics[topic].push({
            token: token,
            func: func
        });
        return token;
    };
    // Unsubscribe from a specific
    // topic, based on a tokenized reference
    // to the subscription
    myObject.unsubscribe = function( token ) {
        for ( var m in topics ) {
            if ( topics[m] ) {
                for ( var i = 0, j = topics[m].length; i < j; i++ ) {
                    if ( topics[m][i].token === token ) {
                        topics[m].splice( i, 1 );
                        return token;
                    }
                }
            }
        }
        return this;
    };
}( pubsub ));
```

十六，实现 lodash 的_.get

示例：get(  { a: { b: 1 } } , 'a.b' ); // 输出 1
```
function get(obj = {} , key){
    return key.split('.').reduce((obj,path) => (obj || {})[path] , {});
}
```

十七，实现 add(1)(2)(3) , add(1,2)(3) // 参数固定
```
function add(...args){
    return args.length === 3 ? args.reduce((a,b)=>a+b) : function (...args1){
        return add( ...(args.concat(args1)) );
    }
}
add(1)(2)(3) // 6
add(1,2)(3) // 6
```

十八，十七，实现 add(1)(2)(3)() , add(1)(2)(3)(4)() // 参数不固定
```
function add(){
    let allArgs = []
    return function temp(...args){
        if(args.length){
            allArgs = [
                ...allArgs,
                ...args
            ];
            return temp;
        }else{
            let res = allArgs.reduce((a,b)=>a+b);
            allArgs = [];
            return res;
        }
    }
}
add(1)(2)(3)() //6
add(1)(2)(3, 4, 5)() //15
add(1)(2, 3, 4, 5)() //15
```

十九，手写promise/promise.all/promise.race

```
class newPromise{
    constructor(fn){
        this.state = 'pending';
        this.value;
        this.reason;
        const resolve = value => {
            if(this.state === 'pending'){
                this.state = 'fulfilled'
                this.value = value;
            }
        }
        const reject = reason => {
            if(this.state === 'pending'){
                this.state = 'reject';
                this.reason = reason;
            }
        }
        try{
            fn(resolve , reject);
        }catch(err){
            reject(err);
        }
    }
    then(onResolve , onReject){
        switch(this.state){
            case 'fulfilled':
                onResolve(this.value);
            break;
            case 'reject':
                onReject(this.reason);
            break;
        }
    }
}

// promise.all
Prmoise.prototype.all = function(list){
    return new Promise((resolve , reject)=>{
        if(typeof list !== 'function'){
            throw error('...');
        }
        let res = [];
        let index = 0;
        list.forEach((e,i)=>{
            Promise.resolve(list[i]).then(value=>{
                res[i] = value;
            });
            index++;
            if(index === list.length){
                resolve(res);
            }
        });
    });
}

// promise.race
Prmoise.prototype.race = function(list){
    return new Promise((resolve , reject)=>{
        if(typeof list !== 'function'){
            throw error('...');
        }
        list.forEach((e,i)=>{
            Promise.resolve(list[i]).then(value=>{
                resolve(value);
            });
        });
    });
}
```

二十，ES6proxy 实现 arr[-1] === arr[arr.length-1]

```
let arr = new Proxy([],{
    get: function(target , key){
        if(key < 0){
            key = target.length + key;    
        }
        return target[key];
        // ES6 reflect 版本
        // return Reflect.get(target , key);
    }
});
```

二十一，手写jsonp实现
```
function jsonp ({url, param, callback}) {
  return new Promise((resolve, reject) => {
    var script = document.createElement('script')
    window.callback = function (data) {
      resolve(data);
    }
    var param = {...param, callback}
    var arr = []
    for (let key in param) {
      arr.push(`${key}=${param[key]}`)
    }
    script.src = `${url}?${arr.join('&')}`
    document.body.appendChild(script)
  })
}
```

二十二，请实现一个cacheRequest方法，保证当使用ajax(请求相同资源时，此题中相同资源的判断是以url为判断依据)，真实网络层中，实际只发出一次请求（假设已存在request方法用于封装ajax请求，调用格式为：```request(url, successCallback, failCallback)```）
比如调用方代码（并行请求）如下
```javascript
cacheRequest('/user', data => {
    console.log('我是从A中请求的user，数据为' + data);
})

cacheRequest('/user', data => {
    console.log('我是从B中请求的user，数据为' + data);
})
```

```javascript
let requestObj = {};
const cacheRequest = (url , successCallback, failCallback) => {
    if(requestObj[url]){
        if(requestObj[url].status){
            // 已经请求过，直接返回结果
            requestObj[url].status === 'success' ? successCallback(requestObj[url].data) : failCallback(requestObj[url].data);
        }else if(requestObj[url].list && requestObj[url].list.length > 0){
            // 已请求，但未返回结果，将未执行函数放入队列
            requestObj[url].list.push({
                success: successCallback,
                fail: failCallback
            });
        }else{
            // 未请求
            requestObj[url].list = [{
                success: successCallback,
                fail: failCallback
            }];
            request(url, data=>{
                let succData = {
                    status: 'success',
                    data: data
                }
                requestObj[url].list.forEach(e=>{
                    e.success(data);
                });
                requestObj[url] = succData;
            }, err=>{
                let failData = {
                    status: 'fail',
                    data: data
                }
                requestObj[url].list.forEach(e=>{
                    e.fail(data);
                });
                requestObj[url] = failData;
            });
        }
    }

}

```





