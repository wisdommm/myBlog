一，filter

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

二，map

Array.prototype.map = function(fn){
    if( !Array.isArray(this) ){
        throw Error('not array');
        return;
    }
    let res = [];
    let arr = this;
    for(let i = 0 ; i < arr.length ; i++){
        let tmp = fn( arr[i] , i , arr);
        res.push(tmp);
    }
    return res;
}

三，call/apply

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

四，new

function newOption(fn,...args){
    if(typeof fn !== 'function'){
        throw Error('not function');
    }
    let obj = {};
    obj._proto_ = fn.prototype;
    let res = fn.apply(obj , args);
    return typeof res === 'object' ? res : obj;
}

五，bind

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

六，防抖
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

七，节流

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

八，deepClone深拷贝

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

九，数组flatten

function flatten(arr){
    if( Array.isArray(arr) )return;
    let res = [];
    for(let i in arr){
        if(arr[i] instanceof Array){
            res = res.concat( flatten( arr[i] ) );
        }else{
            res.push(arr[i]);
        }
    }
    return res;
}

十，实现一系列函数，能够达到如下调用：one(add(two())) 输出 3；two(add(one())) 输出 3。

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

十一，instanceof

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

十二，实现一个EventEmitter，支持on、off、once、emit事件

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

十三，数据双向绑定

es5版，核心是对象描述符
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

es6版，核心是proxy
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








