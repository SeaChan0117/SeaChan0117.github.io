---
title: ES新特性
date: 2021-06-10 20:52:55
categories:
- JavaScript
- 大前端
tags:
- JavaScript
- ES6
- ES2015
cover: https://tse1-mm.cn.bing.net/th/id/OIP-C.CZ0sjpBMOZw23b5CzpnbzwHaEK?pid=ImgDet&rs=1
---
# ECMAScript（ES）：
也是脚本语言，通常被看成是JavaScript的标准化规范，实际上JavaScript是ES的扩展语言，ES提供了基本的语法，比如怎么去定义变量，怎么进行条件判断等。JavaScript则实现了这些语法，并做了相应的扩展，比如在浏览器环境中操作BOM/DOM，在node环境中操作文件等操作。

浏览器环境：JavaScript = ES + Web APIs（BOM+DOM）  
node环境：JavaScript = ES + Node APIs（fs + net + etc.）
<!--more-->
> ECMAScript和JavaScript的关系：  
1996年11月，JavaScript的创造者--Netscape公司，决定将JavaScript提交给国际标准化组织ECMA，希望这门语言能够成为国际标准。  
次年，ECMA发布262号标准文件（ECMA-262）的第一版，规定了浏览器脚本语言的标准，并将这种语言称为ECMAScript，这个版本就是1.0版。该标准一开始就是针对JavaScript语言制定的，但是没有称其为JavaScript，
有两个方面的原因。  
一是商标，JavaScript本身已被Netscape注册为商标。  
二是想体现这门语言的制定者是ECMA，而不是Netscape，这样有利于保证这门语言的开发性和中立性。

# ECMAScript2015（ES2015/ES6）：
## 1.较之前版本的变化
* 解决原有语法上的问题或不足（let，const提供的块级作用域）；
* 对原有语法进行增强（解构展开，模板字符串等）；
* 全新的对象，全新的方法，全新的功能（Promise，proxy等）；
* 全新的数据类型和数据结构（Symbol，Set，Map）；

## 2.let/const与块级作用域
作用域，即某个成员起作用的范围。在全局作用域和函数作用域的基础之上，ES2015又新增了块级作用域。

块，就是代码中一对花括号内的范围，如if语句和for循环语句中的花括号都是块，在没有块级作用域之前，在块里边定义的变量（使用var声明的变量），在外边也能访问到。使用let/const声明的变量在块外边是无法访问到的。

const在let基础上多了一个只读的特性，即变量一旦声明就不可改变（指向内存地址不可改变），所以const声明变量时必须赋值。

如果声明的变量是一个引用，引用中的属性改变不会改变变量的指向地址，是有效的。

```javascript
const obj = {}
obj.name = 'lagou'  // 未改变obj指向地址，有效
obj = {}   // 重新赋值，修改了obj的指向地址，报错
```
**最佳实践：不用var，主用const，配合let**

## 3.数组和对象的解构
解构赋值是对赋值运算符的扩展。它是一种针对数组或者对象进行模式匹配，然后对其中的变量进行赋值。在代码书写上简洁且易读，语义更加清晰明了；也方便了复杂对象中数据字段获取。

### 数组解构
#### 基本
```javascript
let [a, b, c] = [1, 2, 3]; // a = 1// b = 2// c = 3
```
#### 可嵌套
```javascript
let [a, [[b], c]] = [1, [[2], 3]];
// a = 1
// b = 2
// c = 3
```

#### 可忽略
```javascript
let [a, , b] = [1, 2, 3];
// a = 1
// b = 3
```

#### 不完全解构
```javascript
let [a = 1, b] = []; // a = 1, b = undefined
```

#### 剩余运算符
```javascript
let [a, ...b] = [1, 2, 3];
//a = 1
//b = [2, 3]
```

#### 字符串
```javascript
let [a, b, c, d, e] = 'hello';
// a = 'h'
// b = 'e'
// c = 'l'
// d = 'l'
// e = 'o'
```

#### 解构默认值
```javascript
let [a = 2] = [undefined]; // a = 2
let [a = 3, b = a] = [];     // a = 3, b = 3
let [a = 3, b = a] = [1];    // a = 1, b = 1
let [a = 3, b = a] = [1, 2]; // a = 1, b = 2
```

### 对象解构
#### 基本
```javascript
let { foo, bar } = { foo: 'aaa', bar: 'bbb' };
// foo = 'aaa'
// bar = 'bbb'
 
let { baz : foo } = { baz : 'ddd' };
// foo = 'ddd'
```

#### 可嵌套可忽略
```javascript
let obj = {p: ['hello', {y: 'world'}] };
let {p: [x, { y }] } = obj;
// x = 'hello'
// y = 'world'
let obj = {p: ['hello', {y: 'world'}] };
let {p: [x, {  }] } = obj;
// x = 'hello'
```

#### 不完全解构
```javascript
let obj = {p: [{y: 'world'}] };
let {p: [{ y }, x ] } = obj;
// x = undefined
// y = 'world'
```

#### 剩余运算符
```javascript
let {a, b, ...rest} = {a: 10, b: 20, c: 30, d: 40};
// a = 10
// b = 20
// rest = {c: 30, d: 40}
```

#### 解构默认值
```javascript
let {a = 10, b = 5} = {a: 3};
// a = 3; b = 5;
let {a: aa = 10, b: bb = 5} = {a: 3};
// aa = 3; bb = 5;
```

## 4.模板字符串
模板字符串相当于加强版的字符串，用反引号`,除了作为普通字符串，还可以用来定义多行字符串，还可以在字符串中加入变量和表达式。（模板字符串中的换行和空格都是会被保留的）
```javascript
let string1 =  `Hey,
can you stop angry now?`;
console.log(string1);
// Hey,
// can you stop angry now?

let name = "Mike";
let age = 27;
let info = `My Name is ${name},I am ${age+1} years old next year.`
console.log(info);
// My Name is Mike,I am 28 years old next year.

function f(){
  return "have fun!";
}
let string2= `Game start,${f()}`;
console.log(string2);  // Game start,have fun!
```

标签模板----是一个函数的调用，其中调用的参数是模板字符串。
```javascript
alert`Hello world!`;
// 等价于
alert('Hello world!');
```

## 5.字符串的扩展方法（startsWith，endsWith，includes）
## 6.参数默认值
```javascript
function myFunction(x, y = 10) {
    // y is 10 if not passed or undefined
    return x + y;
}
 
myFunction(0, 2) // 输出 2
myFunction(5); // 输出 15, y 参数的默认值
```
## 7.剩余参数
JS函数内部有个arguments对象，可以拿到全部实参，ES6给我们带来了一个新的对象，可以拿到除开始参数外的参数，即剩余参数。
```javascript
function func(a, ...rest) {
  console.log(a)
  console.log(rest)
}
func(1) 
// 1
// []
func(1, 2, 3, 4)
// 1
// [2, 3, 4]

function func(a, ...rest, b) {
 
}
// 报错，剩余参数即只能出现在最后


// 当您使用剩余参数后，函数的length属性会发生一些变化,即length不包含rest
function func(a, b, ...rest) {
}
func.length // 2


// rest不能和arguments一起使用，FireFox会报错
```

arguments和剩余参数的区别
* arguments是一个伪数组（Array-like）
* 剩余参数是一个真正数组（Array），具有Array.prototype上的所有方法
* arguments上有callee，callee上有caller

## 8.数组展开
其实参照上一条，可以看出“...”符号的作用是将数组展开。
```javascript
const arr = [1, 2, 3]
console.log(...arr)
// 1 2 3
```

## 9.箭头函数
极大地减少代码量，简化代码且更易读；
```javascript
function a(n1, n2) {
   return n1+n2
}

// =>

let a = (n1, n2) => n1 + n2
```

箭头函数内部的this是词法作用域，由上下文确定。
```javascript
const person = {
  name: 'Tom',
  say: () => {
    console.log(this.name)
  },
  say_: function() {
    setTimeout(()=>{
      console.log(this.name)
    })
  }
}

person.say()
// undefined

person.say_()
// Tom
```

## 10.Object.assign
Object.assign方法用来将源对象（source）的所有可枚举属性，复制到目标对象（target）。它至少需要两个对象作为参数，第一个参数是目标对象，后面的参数都是源对象。只要有一个参数不是对象，就会抛出TypeError错误。

如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。
```javascript
var target  =  { a: 1 };
var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}

// 如果只有一个参数，Object.assign会直接返回该参数。
var obj = {a: 1};  
Object.assign(obj) === obj // true  

// 如果该参数不是对象，则会先转成对象，然后返回。
typeof Object.assign(2) // "object"  

// 由于undefined和null无法转成对象，所以如果它们作为参数，就会报错。
Object.assign(undefined) //  报错  
Object.assign(null) //  报错  

// Object.assign只拷贝自身属性，不可枚举的属性（enumerable为false）和继承的属性不会被拷贝。
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  })
)
// { b: 'c' }


Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: true,
    value: 'hello'
  })
)
// {b: "c", invisible: "hello"}

// 对于嵌套的对象，Object.assign的处理方法是替换，而不是添加。
var target = { a: { b: 'c', d: 'e' } }
var source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }

// Object.assign方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。
var obj1 = {a: {b: 1}};  
var obj2 = Object.assign({}, obj1);  
obj1.a.b = 2;  
obj2.a.b // 2
```

11.Proxy（大部分摘自：http://blog.poetries.top/2018/12/21/es6-proxy/）
proxy在目标对象的外层搭建了一层拦截，外界对目标对象的某些操作，必须通过这层拦截。
```javascript
var proxy = new Proxy(target, handler);
```

new Proxy()表示生成一个Proxy实例，target参数表示所要拦截的目标对象，handler参数也是一个对象，用来定制拦截行为。
```javascript
var target = {
   name: 'poetries'
 };
 var logHandler = {
   get: function(target, key) {
     console.log(`${key} 被读取`);
     return target[key];
   },
   set: function(target, key, value) {
     console.log(`${key} 被设置为 ${value}`);
     target[key] = value;
   }
 }
 var targetWithLog = new Proxy(target, logHandler);
 
 targetWithLog.name; // 控制台输出：name 被读取
 targetWithLog.name = 'others'; // 控制台输出：name 被设置为 others
 
 console.log(target.name); // 控制台输出: others
```

* targetWithLog 读取属性的值时，实际上执行的是 logHandler.get ：在控制台输出信息，并且读取被代理对象 target 的属性。
* 在 targetWithLog 设置属性值时，实际上执行的是 logHandler.set ：在控制台输出信息，并且设置被代理对象 target 的属性的值
```javascript
// 由于拦截函数总是返回35，所以访问任何属性都得到35
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

Proxy 实例也可以作为其他对象的原型对象
```javascript
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```

proxy对象是obj对象的原型，obj对象本身并没有time属性，所以根据原型链，会在proxy对象上读取该属性，导致被拦截。

对于代理模式Proxy的作用主要体现在三个方面：

拦截和监视外部对对象的访问
降低函数或类的复杂度
在复杂操作前对操作进行校验或对所需资源进行管理
Proxy所能代理的范围--handler

实际上handler本身就是ES6所新设计的一个对象.它的作用就是用来 自定义代理对象的各种可代理操作 。它本身一共有13中方法,每种方法都可以代理一种操作.其13种方法如下：
```javascript
// 在读取代理对象的原型时触发该操作，比如在执行 Object.getPrototypeOf(proxy) 时。
handler.getPrototypeOf()

// 在设置代理对象的原型时触发该操作，比如在执行 Object.setPrototypeOf(proxy, null) 时。
handler.setPrototypeOf()

// 在判断一个代理对象是否是可扩展时触发该操作，比如在执行 Object.isExtensible(proxy) 时。
handler.isExtensible()

// 在让一个代理对象不可扩展时触发该操作，比如在执行 Object.preventExtensions(proxy) 时。
handler.preventExtensions()

// 在获取代理对象某个属性的属性描述时触发该操作，比如在执行 Object.getOwnPropertyDescriptor(proxy, "foo") 时。
handler.getOwnPropertyDescriptor()

// 在定义代理对象某个属性时的属性描述时触发该操作，比如在执行 Object.defineProperty(proxy, "foo", {}) 时。
handler.defineProperty()
 
// 在判断代理对象是否拥有某个属性时触发该操作，比如在执行 "foo" in proxy 时。
handler.has()

// 在读取代理对象的某个属性时触发该操作，比如在执行 proxy.foo 时。
handler.get()

// 在给代理对象的某个属性赋值时触发该操作，比如在执行 proxy.foo = 1 时。
handler.set()

// 在删除代理对象的某个属性时触发该操作，比如在执行 delete proxy.foo 时。
handler.deleteProperty()

// 在获取代理对象的所有属性键时触发该操作，比如在执行 Object.getOwnPropertyNames(proxy) 时。
handler.ownKeys()

// 在调用一个目标对象为函数的代理对象时触发该操作，比如在执行 proxy() 时。
handler.apply()

// 在给一个目标对象为构造函数的代理对象构造实例时触发该操作，比如在执行new proxy() 时。
handler.construct()
```

### Proxy场景
实现私有变量
```javascript
var target = {
   name: 'poetries',
   _age: 22
}

var logHandler = {
  get: function(target,key){
    if(key.startsWith('_')){
      console.log('私有变量age不能被访问')
      return false
    }
    return target[key];
  },
  set: function(target, key, value) {
     if(key.startsWith('_')){
      console.log('私有变量age不能被修改')
      return false
    }
     target[key] = value;
   }
} 
var targetWithLog = new Proxy(target, logHandler);
 
// 私有变量age不能被访问
targetWithLog.name; 
 
// 私有变量age不能被修改
targetWithLog.name = 'others';
```
 
在下面的代码中，我们声明了一个私有的apiKey，便于api这个对象内部的方法调用，但不希望从外部也能够访问api._apiKey
```javascript
var api = {  
    _apiKey: '123abc456def',
    /* mock methods that use this._apiKey */
    getUsers: function(){}, 
    getUser: function(userId){}, 
    setUser: function(userId, config){}
};

// logs '123abc456def';
console.log("An apiKey we want to keep private", api._apiKey);

// get and mutate _apiKeys as desired
var apiKey = api._apiKey;  
api._apiKey = '987654321';
```

很显然，约定俗成是没有束缚力的。使用 ES6 Proxy 我们就可以实现真实的私有变量了，下面针对不同的读取方式演示两个不同的私有化方法。第一种方法是使用 set / get 拦截读写请求并返回 undefined:
```javascript
let api = {  
    _apiKey: '123abc456def',
    getUsers: function(){ }, 
    getUser: function(userId){ }, 
    setUser: function(userId, config){ }
};

const RESTRICTED = ['_apiKey'];
api = new Proxy(api, {  
    get(target, key, proxy) {
        if(RESTRICTED.indexOf(key) > -1) {
            throw Error(`${key} is restricted. Please see api documentation for further info.`);
        }
        return Reflect.get(target, key, proxy);
    },
    set(target, key, value, proxy) {
        if(RESTRICTED.indexOf(key) > -1) {
            throw Error(`${key} is restricted. Please see api documentation for further info.`);
        }
        return Reflect.get(target, key, value, proxy);
    }
});

// 以下操作都会抛出错误
console.log(api._apiKey);
api._apiKey = '987654321'; 
``` 

第二种方法是使用has拦截in操作
```javascript
var api = {  
    _apiKey: '123abc456def',
    getUsers: function(){ }, 
    getUser: function(userId){ }, 
    setUser: function(userId, config){ }
};

const RESTRICTED = ['_apiKey'];
api = new Proxy(api, {  
    has(target, key) {
        return (RESTRICTED.indexOf(key) > -1) ?
            false :
            Reflect.has(target, key);
    }
});

// these log false, and `for in` iterators will ignore _apiKey
console.log("_apiKey" in api);

for (var key in api) {  
    if (api.hasOwnProperty(key) && key === "_apiKey") {
        console.log("This will never be logged because the proxy obscures _apiKey...")
    }
}
```

抽离校验模块
> 让我们从一个简单的类型校验开始做起，这个示例演示了如何使用Proxy保障数据类型的准确性
```javascript
let numericDataStore = {  
    count: 0,
    amount: 1234,
    total: 14
};

numericDataStore = new Proxy(numericDataStore, {  
    set(target, key, value, proxy) {
        if (typeof value !== 'number') {
            throw Error("Properties in numericDataStore can only be numbers");
        }
        return Reflect.set(target, key, value, proxy);
    }
});

// 抛出错误，因为 "foo" 不是数值
numericDataStore.count = "foo";

// 赋值成功
numericDataStore.count = 333;
```


> 如果要直接为对象的所有属性开发一个校验器可能很快就会让代码结构变得臃肿，使用Proxy则可以将校验器从核心逻辑分离出来自成一体
```javascript
function createValidator(target, validator) {  
    return new Proxy(target, {
        _validator: validator,
        set(target, key, value, proxy) {
            if (target.hasOwnProperty(key)) {
                let validator = this._validator[key];
                if (!!validator(value)) {
                    return Reflect.set(target, key, value, proxy);
                } else {
                    throw Error(`Cannot set ${key} to ${value}. Invalid.`);
                }
            } else {
                throw Error(`${key} is not a valid property`)
            }
        }
    });
}

const personValidators = {  
    name(val) {
        return typeof val === 'string';
    },
    age(val) {
        return typeof age === 'number' && age > 18;
    }
}
class Person {  
    constructor(name, age) {
        this.name = name;
        this.age = age;
        return createValidator(this, personValidators);
    }
}

const bill = new Person('Bill', 25);

// 以下操作都会报错
bill.name = 0;  
bill.age = 'Bill';  
bill.age = 15;  
```

> 通过校验器和主逻辑的分离，你可以无限扩展 personValidators 校验器的内容，而不会对相关的类或函数造成直接破坏。更复杂一点，我们还可以使用 Proxy 模拟类型检查，检查函数是否接收了类型和数量都正确的参数
```javascript
let obj = {  
    pickyMethodOne: function(obj, str, num) { /* ... */ },
    pickyMethodTwo: function(num, obj) { /*... */ }
};

const argTypes = {  
    pickyMethodOne: ["object", "string", "number"],
    pickyMethodTwo: ["number", "object"]
};

objProxy = new Proxy(obj, {  
    get: function(target, key, proxy) {
        var value = target[key];
        return function(...args) {
            var checkArgs = argChecker(key, args, argTypes[key]);
            return Reflect.apply(value, target, args);
        };
    }
});

function argChecker(name, args, checkers) {  
    for (var idx = 0; idx < args.length; idx++) {
        var arg = args[idx];
        var type = checkers[idx];
        if (!arg || typeof arg !== type) {
            console.warn(`You are incorrectly implementing the signature of ${name}. Check param ${idx + 1}`);
        }
    }
}

objProxy.pickyMethodOne(1,1,'');  
// > You are incorrectly implementing the signature of pickyMethodOne. Check param 1
// > You are incorrectly implementing the signature of pickyMethodOne. Check param 2
// > You are incorrectly implementing the signature of pickyMethodOne. Check param 3

objProxy.pickyMethodTwo("wopdopadoo", {});  
// > You are incorrectly implementing the signature of pickyMethodTwo. Check param 1

// No warnings logged
objProxy.pickyMethodOne({}, "a little string", 123);  
objProxy.pickyMethodTwo(123, {});
```

访问日志
> 对于那些调用频繁、运行缓慢或占用执行环境资源较多的属性或接口，开发者会希望记录它们的使用情况或性能表现，这个时候就可以使用Proxy充当中间件的角色，轻而易举实现日志功能
```javascript
let api = {  
    _apiKey: '123abc456def',
    getUsers: function() { /* ... */ },
    getUser: function(userId) { /* ... */ },
    setUser: function(userId, config) { /* ... */ }
};

function logMethodAsync(timestamp, method) {  
    setTimeout(function() {
        console.log(`${timestamp} - Logging ${method} request asynchronously.`);
    }, 0)
}

api = new Proxy(api, {  
    get: function(target, key, proxy) {
        var value = target[key];
        return function(...arguments) {
            logMethodAsync(new Date(), key);
            return Reflect.apply(value, target, arguments);
        };
    }
});

api.getUsers();
```

预警和拦截

> 假设你不想让其他开发者删除 noDelete 属性，还想让调用 oldMethod 的开发者了解到这个方法已经被废弃了，或者告诉开发者不要修改 doNotChange 属性，那么就可以使用 Proxy 来实现
```javascript
let dataStore = {  
    noDelete: 1235,
    oldMethod: function() {/*...*/ },
    doNotChange: "tried and true"
};

const NODELETE = ['noDelete'];  
const NOCHANGE = ['doNotChange'];
const DEPRECATED = ['oldMethod'];  

dataStore = new Proxy(dataStore, {  
    set(target, key, value, proxy) {
        if (NOCHANGE.includes(key)) {
            throw Error(`Error! ${key} is immutable.`);
        }
        return Reflect.set(target, key, value, proxy);
    },
    deleteProperty(target, key) {
        if (NODELETE.includes(key)) {
            throw Error(`Error! ${key} cannot be deleted.`);
        }
        return Reflect.deleteProperty(target, key);

    },
    get(target, key, proxy) {
        if (DEPRECATED.includes(key)) {
            console.warn(`Warning! ${key} is deprecated.`);
        }
        var val = target[key];

        return typeof val === 'function' ?
            function(...args) {
                Reflect.apply(target[key], target, args);
            } :
            val;
    }
});

// these will throw errors or log warnings, respectively
dataStore.doNotChange = "foo";  
delete dataStore.noDelete;  
dataStore.oldMethod();
```

过滤操作

> 某些操作会非常占用资源，比如传输大文件，这个时候如果文件已经在分块发送了，就不需要在对新的请求作出响应（非绝对），这个时候就可以使用 Proxy 对当请求进行特征检测，并根据特征过滤出哪些是不需要响应的，哪些是需要响应的。下面的代码简单演示了过滤特征的方式，并不是完整代码，相信大家会理解其中的妙处
```javascript
let obj = {  
    getGiantFile: function(fileId) {/*...*/ }
};

obj = new Proxy(obj, {  
    get(target, key, proxy) {
        return function(...args) {
            const id = args[0];
            let isEnroute = checkEnroute(id);
            let isDownloading = checkStatus(id);      
            let cached = getCached(id);

            if (isEnroute || isDownloading) {
                return false;
            }
            if (cached) {
                return cached;
            }
            return Reflect.apply(target[key], target, args);
        }
    }
});
```

中断代理
> Proxy支持随时取消对target的代理，这一操作常用于完全封闭对数据或接口的访问。在下面的示例中，我们使用了Proxy.revocable方法创建了可撤销代理的代理对象：
```javascript
let target = {};
let handler = {};
let {proxy,revoke} = Proxy.revocable(target, handler);
proxy.foo = 123;
proxy.foo // 123
revoke();
proxy.foo // TypeError: Revoked
```

## Proxy和Object.defineProperty()的区别

Object.defineProperty() 的问题主要有三个：  
* 不能监听数组的变化
* 必须遍历对象的每个属性
* 必须深层遍历嵌套的对象
Proxy：  
* 针对对象：针对整个对象,而不是对象的某个属性
* 支持数组：不需要对数组的方法进行重载，省去了众多 hack
* 嵌套支持：get 里面递归调用 Proxy 并返回
> 优势：Proxy 的第二个参数可以有 13 种拦截方法，比 Object.defineProperty() 要更加丰富,Proxy 作为新标准受到浏览器厂商的重点关注和性能优化，相比之下 Object.defineProperty() 是一个已有的老方法。  
> 劣势：Proxy 的兼容性不如 Object.defineProperty() (caniuse 的数据表明，QQ 浏览器和百度浏览器并不支持 Proxy，这对国内移动开发来说估计无法接受，但两者都支持 Object.defineProperty()),不能使用 polyfill 来处理兼容性
>
## 12.Reflect
Reflect是一个内置的对象，它提供拦截 JavaScript 操作的方法。这些方法与proxy handlers的方法相同。Reflect不是一个函数对象，因此它是不可构造的（不能使用new进行调用，所有属性和方法都是静态的）。
```javascript
Reflect.apply(target, thisArgument, argumentsList)
// 对一个函数进行调用操作，同时可以传入一个数组作为调用参数。和 Function.prototype.apply() 功能类似。
Reflect.construct(target, argumentsList[, newTarget])
// 对构造函数进行 new 操作，相当于执行 new target(...args)。
Reflect.defineProperty(target, propertyKey, attributes)
// 和 Object.defineProperty() 类似。如果设置成功就会返回 true
Reflect.deleteProperty(target, propertyKey)
// 作为函数的delete操作符，相当于执行 delete target[name]。
Reflect.get(target, propertyKey[, receiver])
// 获取对象身上某个属性的值，类似于 target[name]。
Reflect.getOwnPropertyDescriptor(target, propertyKey)
// 类似于 Object.getOwnPropertyDescriptor()。如果对象中存在该属性，则返回对应的属性描述符,  否则返回 undefined.
Reflect.getPrototypeOf(target)
// 类似于 Object.getPrototypeOf()。
Reflect.has(target, propertyKey)
// 判断一个对象是否存在某个属性，和 in 运算符 的功能完全相同。
Reflect.isExtensible(target)
// 类似于 Object.isExtensible().
Reflect.ownKeys(target)
// 返回一个包含所有自身属性（不包含继承属性）的数组。(类似于 Object.keys(), 但不会受enumerable影响).
Reflect.preventExtensions(target)
// 类似于 Object.preventExtensions()。返回一个Boolean。
Reflect.set(target, propertyKey, value[, receiver])
// 将值分配给属性的函数。返回一个Boolean，如果更新成功，则返回true。
Reflect.setPrototypeOf(target, prototype)
// 设置对象原型的函数. 返回一个 Boolean， 如果更新成功，则返回true。
```
[更多](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/tugenhua0707/p/10291909.html)

## 13.Promise
在promise这种编程概念出现之前，回调函数是异步编程的主要思想，但是当一个业务依赖另一个业务结果时，就出现了回调嵌套，当回调嵌套越来越多，就形成了回调地狱，代码冗杂，不易读，不易维护，之后便出现了promise的编程概念，直到被ES规范实现成为标准API。

顾名思义，promise就是承诺，这里就是Promise对象对使用者的一个承诺，即当你使用promise去处理一个业务时，它能承诺给你对应的结果，不管是成功还是失败，都是可预见的。且promise实现了链式调用和promise.all等方法，彻底解决了回调地狱的问题，代码层次清晰，更易读，更易维护。

此文不再对Promise做更多详细介绍，具体可参考以下链接：

* 使用 promises —— MDN
* Promise —— MDN
* Promise — 廖雪峰
* JavaScript Promise：去而复返 —— 司徒正美
* (上面的原文)JavaScript Promise：简介 —— Web Fundamentals
* 1 分钟读完《10 分钟学会 JavaScript 的 Async/Await》 —— justjavac
* JavaScript Promise 迷你书（中文版）
* JavaScript 进阶之路——认识和使用 Promise，重构你的 Js 代码 —— 博客园  
熟悉掌握Promise后，可自己尝试实现Promise的一些基本功能，以下给出一些思路和待实现功能点，不详细之处可自行实现：
```javascript
/**
 * 1.Promise是一个类，新建时传递一个立即执行的执行器
 * 2.promise中有三种状态，等待（pending），成功（fulfilled），失败（rejected）
 *      pending --> fulfilled;
 *      pending --> rejected;
 *      状态只能由等待变为成功或者等待变为失败，且一旦状态确定就不可更改
 * 3.resolve和reject方法是用来更改状态的，resolve -> fulfilled，reject -> rejected
 * 4.then方法的作用就是判断状态，如果成功就调用resolve，失败就调用reject，then方法是定义在原型对象中的
 * 5.then的回调函数带有参数，成功时表示成功的值，失败时表示失败的原因
 * 6.如果promise内执行的是异步代码时（异步后resolve或reject），如果直接调用then中的回调，导致无法获得执行结果，所以在then时先存储回调，在真正完成（resolve或reject）的时候再调用相应的回调
 * 7.如果promise的then多次调用应该添加多个处理函数的回调，此时需要将上一步存储的处理函数改造成集合
 * 8.then方法的链式调用，如果要链式调用，then方法应该返回一个promise对象
 * 9.错误捕获，执行器函数和then回调中
 * 10.将then方法中的参数改为可选项
 * 11.实现promise.all方法（全部成功才resolve，一个失败就reject）
 * 12.实现promise.resolve，如果接收的参数是promise了直接返回，不是则new一个然后resolve对应的参数
 * 13.finally方法的实现
 * 14.catch方法实现
 * 15.race方法实现，不管成功还是失败，先结束的先resolve或reject
 */
```

## 14.class类
MDN中对class的定义是，class 声明创建一个基于原型继承的具有给定名称的新类。

简单点，你就理解为，class定义后，可以new出一个实例来，它其实是一个让JavaScript看上去更像面向对象的语法糖。
```javascript
// function和class定义类，都可以new出实例来，即以下方法一和方法二等价
/* 方法一
function Person(name) {
    this.name = name
}

Person.prototype.say = function () {
    console.log(`my name is ${this.name}`);
}
*/

/* 方法二
class Person{
    constructor(name) {
        this.name = name
    }
    
    say() {
        console.log(`my name is ${this.name}`);
    }
}
*/

let p = new Person('SeaChan')
p.say(); // 方法一和方法二都打印相同结果：my name is SeaChan
```

class中的静态方法（static）;不使用new就可通过类名调用的方法，如果其中返回实例，就可不用new直接创建一个实例对象，如下：
```javascript
class Person{
    constructor(name) {
        this.name = name
    }

    say() {
        console.log(`my name is ${this.name}`);
    }

    static create(name) {
        return new Person(name)
    }
}
let p = Person.create('SeaChan')
p.say(); // my name is SeaChan
```

类的继承（extends）；
```javascript
class Person{
    constructor(name) {
        this.name = name
    }

    say() {
        console.log(`my name is ${this.name}`);
    }


}
// 继承Person
class Kids extends Person {
    constructor(name, age) {
        super(name);  // 调用父类的构造函数；
        this.age = age
    }

    hello() {
        super.say();  // 调用父类的say方法；
        console.log(`hello, I am ${this.age} years old!`);
    }
}

let kid = new Kids('SeaChan', 17)
kid.hello()
// my name is SeaChan
// hello, I am 17 years old!
```

## 15.Set和Map
Set和Map是ES6新增的数据类型；

Set和数组很像，它是一系列数据的集合，可以按照插入的顺序迭代它的元素。Set中的元素只会出现一次，即Set中的元素是唯一的。
```javascript
let set = new Set()
set.add(1).add(3).add(2).add(4).add(2)
console.log(set);
// Set(4) {1, 3, 2, 4}

// 遍历set
set.forEach(val => console.log(val))
for (let val of set) {
    console.log(val)
}

console.log(set.size);  // 4 获取集合的长度，类似数组的arr.length；
console.log(set.has(5));    // false 判断集合中是否含有某一个元素，类似数组的arr.includes()
console.log(set.delete(2)); // true 删除集合中的某个元素
console.log(set);           // Set(3) {1, 3, 4}
set.clear()                 // 清空集合
console.log(set);           // Set(0) {}


// 数组和Set之间的转换
const arr = [1, 2, 3, 1, 2, 6, 4]
const res = new Set(arr)
console.log(res)    // Set(5) {1, 2, 3, 6, 4}; 元素唯一性

Array.from(res) // (5) [1, 2, 3, 6, 4];  Set转数组
[...res]        // (5) [1, 2, 3, 6, 4];  Set转数组
```

Map是一种能根据原始插入顺序（即：可迭代）记录键值对的数据类型。其中任何值（对象或者原始值）都可以作为一个键或者一个值。
```javascript
let map = new Map()
map.set('SeaChan', '99')
map.set('Tom', '90')
let key1 = {a: 123}
map.set(key1, 'hahahaha')
console.log(map);
// map.get('SeaChan')   根据键读取值
// map.has('SeaChan')   判断是否存在某个键值对
// map.delete('Tom')    删除某个键值对
// map.clear()          清空map

map.forEach((val, key) => {
    console.log(key, val)
})
```

## 16.Symbol
Symbol是ES6新增的基本数据类型，每个从Symbol()返回的symbol值都是唯一的，使用它的目的是为对象创建一个唯一属性的标识符，保证对象中的属性不会被修改或者被覆盖。
```javascript
const sym1 = Symbol()
const sym2 = Symbol('foo')
const sym3 = Symbol('foo')

sym2 === sym3    // false

// Symbol 数据类型具有隐藏性，for···in，object.keys()不能访问
let obj = {
    key0: '000',
    [sym1]: '123',
    [sym2]: '456'
}
for (const objKey in obj) {
    console.log(objKey);
}
// key0  只打印一个key0
console.log(Object.keys(obj));  // ["key0"]

// 要访问对象中存在的symbol属性，可以使用Object.getOwnPropertySymbols
console.log(Object.getOwnPropertySymbols(obj))   // (2) [Symbol(), Symbol(foo)]

// 为了保证多次使用同一个symbol的情况，可以使用全局注册并登记的方法：Symbol.for()
const s1 = Symbol.for('foo')
const s2 = Symbol.for('foo')
console.log(s1 === s2);     // true
// 注意： Symbol.for()接收一个字符串，如果不是字符串会转为字符串，即Symbol.for('true') === Symbol.for(true)

// 反之能通过另一个方法获取对象的参数值，Symbol.keyFor()
 console.log(Symbol.keyFor(s1));  // 'foo'
 console.log(Symbol.keyFor(s2));  // 'foo'

// JSON.stringfy()无法序列化对象中的symbol属性，会被忽略
JSON.stringify(obj)   // "{"key0":"000"}"
```

## 17.for...of循环
因为for...in对象遍历存在一些一定的局限性，引入了全新的for...of循环，作为遍历所有数据结构的统一方式。
```javascript
const arr = [1,2,3,4]
// for (const item of arr) {
//     console.log(item);
// }

// arr.forEach(item => {
//     console.log(item)
// })

for (const item of arr) {
    if (item > 2) {
        break           // 可跳出循环，arr.forEach()不可以
    }
}

// 遍历Set和Map等实现了iterator接口的数据
const s = new Set(['s1', 's2', 's3'])
for (const item of s) {
    console.log(item);
}
// s1 s2 s3

const m = new Map()
m.set(Symbol.for('m1'), 'm1')
m.set(Symbol.for('m2'), 'm2')

for (const [val, key] of m) {
    console.log(key, val);
}

// (2) Symbol(m1) "m1"
// (2) Symbol(m2) "m2"


// 让我们试试原始的Object使用for...of会怎样
const obj = {
    a: 1,
    b: 2
}

for (const item of obj) {
    console.log(item);
}
// 报错： Uncaught TypeError: obj is not iterable
```

以上看到Object通过for...of遍历报错，提示obj不可迭代，说明要满足for...of循环必须满足iterator条件，为了给数据提供这种条件，ES2015提供了iterator接口，实现该接口就是for...of遍历的基础。来看一下数组上的iterator：
```javascript
const arr = [1,2,3]
let ire = arr[Symbol.iterator]()
ire.next()   // {value: 1, done: false}
ire.next()   // {value: 2, done: false}
ire.next()   // {value: 3, done: false}
ire.next()   // {value: undefined, done: true}
```

可迭代接口实现：
```javascript
const obj = {
    store: [1, 2, 3, 4],
    [Symbol.iterator]: function () {
        let index = 0
        const _this = this
        return {
            next: function () {
                return {
                    value: _this.store[index],
                    done: index++ >= _this.store.length
                }
            }
        }
    }
}

for (const item of obj) {
    console.log(item);
}
// 1 2 3 4
```

迭代器模式：
```javascript
/**迭代器设计模式*/
// 场景：你我协同开发一个任务清单应用
// A的代码：=============================
/*const todos = {
        life: ['吃饭', '睡觉', '打豆豆'],
        learn: ['语文', '数学', '英语']
    }*/

    // =====>  todos中每次添加一个集合，B就要添加一次遍历，且还需要沟通是啥字段
// 修改为迭代器模式
const todos = {
    life: ['吃饭', '睡觉', '打豆豆'],
    learn: ['语文', '数学', '英语'],
    work: ['编程', '洗衣服'],
    [Symbol.iterator]: function () {
        let index = 0
        const allData = [...this.life, ...this.learn, ...this.work]
        return {
            next: function () {
                return {
                    value: allData[index],
                    done: index++ >= allData.length
                }
            }
        }
    }
}


// B的代码：=============================
// 要获取所有todo的项目
/*for (const item of todos.life) {
    console.log(item);
}
for (const item of todos.learn) {
    console.log(item);
}*/

// A使用迭代器模式后 ======>
for (const item of todos) {
    console.log(item);  // 吃饭 睡觉 打豆豆 语文 数学 英语 编程 洗衣服
}
```

## 18.生成器（Generator）
由生成器函数函数返回，并且满足可迭代接口。

避免异步编程中回调嵌套太深，提供更好的异步编程解决方案。
```javascript
// Generator 函数
function * fn() {
    console.log('SeaChan');
}

let f = fn()
console.log(f);   // 打印一个生成器对象

f.next()
// SeaChan
// {value: 123, done: true}

f.next()
// {value: undefined, done: true}
```

完整的生成器配合yield使用，实质效果就是，调用一下生成器对象的next方法，就执行一部分代码，到下一个yield停止，next直到所有的yield执行完后结果变为{value: undefined, done: true}不再改变。
```javascript
function * fn() {
    yield 1
    yield 2
    yield 3
}

const g = fn()
g.next() // {value: 1, done: false}
g.next() // {value: 2, done: false}
g.next() // {value: 3, done: false}
g.next() // {value: undefined, done: true}
```

Generator应用：
```javascript
// 发号器，执行一次next方法，发号器号码自增1
function *numberMaker() {
    let i = 0
    while (true) {
        yield i++
    }
}

let numMaker = numberMaker()
console.log(numMaker.next().value);  // 0
console.log(numMaker.next().value);  // 1
console.log(numMaker.next().value);  // 2
......


// 使用生成器函数实现 iterator 接口
const todos = {
    life: ['吃饭', '睡觉', '打豆豆'],
    learn: ['语文', '数学', '英语'],
    work: ['编程', '洗衣服'],
    [Symbol.iterator]: function * () {
        const allData = [...this.life, ...this.learn, ...this.work]
        for (const item of allData) {
            yield item
        }
    }
}
for (const item of todos) {
    console.log(item);      // 吃饭 睡觉 打豆豆 语文 数学 英语 编程 洗衣服
}
```

## 19.ES2016 新增特性
```javascript
// ES2016
// Array.prototype.includes()  判断数组是否含有某一个元素
const arr = [1,'seachan', NaN]
arr.includes(1)          // true
arr.includes('seachan')  // true
arr.includes(NaN)        // true
arr.includes(0)          // false


// 指数运算符 ** ； num1 ** num2 表示num1的num2次幂
// 2**3 ====> 8
```

## 20.ES2017 新增特性
```javascript
const obj = {
    a: 1,
    b: 2
}
// Object.values() 获取对象中值的集合
Object.values(obj)  // [1, 2]

// Object.entries()方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for...in 循环遍历该对象时返回的顺序一致（区别在于 for-in 循环还会枚举原型链中的属性）。
for (const [key,val] of Object.entries(obj)) {
    console.log(key, val);  
}
new Map(Object.entries(obj))

// a 1
// b 2
// Map(2) {"a" => 1, "b" => 2}

// Object.getOwnPropertyDescriptors()  方法用来获取一个对象的所有自身属性的描述符
const p1 = {
    firstName: 'Sea',
    lastName: 'Chan',
    get fullName() {
        return this.firstName + ' ' + this.lastName
    }
}
console.log(p1.fullName)            // Sea Chan
const p2 = Object.assign({}, p1)
p2.lastName = 'Li'
console.log(p2.fullName)            // Sea Chan
let des = Object.getOwnPropertyDescriptors(p1)
const p3 = Object.defineProperties({}, des)
p3.lastName = 'Li'
console.log(p3.fullName);           // Sea Li

// String.prototype.padStart()和String.prototype.padEnd()   在字符串前和字符串后添加某个字符知道满足某个长度
const str = 'string'
str.padStart(10, '+')  // "++++string"
str.padEnd(10, '@')    // "string@@@@"

// 在函数参数中添加尾逗号，利于代码管理工具判断修改的代码
function fn(param1,
param2,
) {
   //***
}
// Async/Await
```



大前端，冲鸭！
