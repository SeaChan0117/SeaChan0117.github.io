---
title: JavaScript 性能优化
date: 2021-06-11 09:44:23
categories:
- JavaScript
tags:
- JavaScript
---
找工作的小伙伴是否有注意到，现在招聘要求中基本都会涉及性能优化的部分？  
前端涉及优化点很多，本文重点关注JavaScript语言层面能做的优化，今天我们就来让 JavaScript 搞快点，从以下几点展开介绍：

* 内存管理
* 垃圾回收与常见的GC算法
* V8引擎的垃圾回收
* Performance工具
* 代码优化实例  

# JavaScript内存管理（Memory Management）
## 内存管理介绍

* 内存：由可读写单元组成，表示一片可操作空间
* 管理：人为地去操作一片空间的申请、使用和释放
* 内存管理：开发者主动申请空间、使用空间、释放空间
* 管理流程：申请 --> 使用 --> 释放

## JavaScript中的内存管理
JavaScript中没有提供API供开发者主动调用操作内存空间，但是js中的一个生命周期也经历了这几个阶段：
```javascript
// 申请 ===================
let obj = {}    // 变量声明时，分配内存
 
// 使用 ===================
obj.name = 'SeaChan'    // 变量的读写操作，操作内存过程
 
// 释放 ===================
obj = null            // 变量置空，释放内存
```

# JavaScript中的垃圾回收机制
JavaScript中内存管理是**自动执行**的，是不可见的，对象不再被引用时是垃圾，对象不能从根上访问到时是垃圾，JavaScript会自发的发现并清理它们。

**可达对象**：可以访问到的对象就是可达对象（引用，或作用域链），可达的标准就是**从根出发**是否能被找到，JavaScript中的根可以理解为全局变量对象（全局执行上下文）。
```javascript
let obj = {name: 'SeaChan'}
let obj1 = obj
obj = null
console.log(obj1)   // {name: "SeaChan"} ==> obj1可达

// ==============================================================
function objGroup(obj1, obj2) {
    obj1.next = obj2
    obj2.prev = obj1

    return {
        o1: obj1,
        o2: obj2
    }
}

let obj = objGroup({name: 'obj1'}, {name: 'obj2'})
console.log(obj);
// node 运行结果
// {
//     o1: <ref *1> {
//     name: 'obj1',
//     next: { name: 'obj2', prev: [Circular *1] }
// },
//     o2: <ref *2> {
//     name: 'obj2',
//     prev: <ref *1> { name: 'obj1', next: [Circular *2] }
// }
// }
```
![图1](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/1.jpg)

![图2](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/2.jpg)
# GC（Garbage Collection）算法介绍
## GC定义与作用
* GC就是垃圾回收机制的简写；
* GC可以找到内存中的垃圾，并释放和回收空间；
```javascript
// GC 里垃圾是什么
// 程序中不再使用的对象
function foo() {
    name = 'SeaChan'            // 全局变量，函数调用完后，不再被使用
    return name
}

foo()

// 程序中不能再访问到的对象
function foo() {
    const name = 'SeaChan'      // 块级作用域局部变量，函数调用完后，不能再访问到
    return name
}

foo()
```

## GC算法是什么
* GC是一种机制，垃圾回收器完成具体的工作；
* 工作内容就是查找垃圾释放空间，回收空间；
* 算法就是工作时查找和回收所遵循的规则；

## 常见的GC算法
* 引用计数
* 标记清除
* 标记整理
* 分代回收

# 引用计数算法
通过设置引用数（引用计数器），引用关系改变时，修改引用数值，判断当前引用数是否为0，为0时立即进行回收。
```javascript
let a = {name: 'SeaChan'}   // 对象被 a 引用，引用 +1  count： 1
a = null                    // 对象引用 -1            count: 0   ====>  将被回收
```

## 引用计数算法优点：
* 发现垃圾时立即回收；（因为引用计数的实时性，能尽快找到垃圾并回收）
* 最大程度减少程序暂停；（当内存爆满时，能立刻找到引用计数为0的对象进行释放，保证内存充足，减少程序暂停）

## 引用计数算法缺点：
* 无法回收循环引用的对象；
* 时间开销大；（实时对引用进行监控，相对于其他GC算法要消耗更多时间）
```javascript
// 循环引用的对象
// 在foo局部作用域中，foo调用完后，obj1和obj2是垃圾，但是他们之间互相引用导致两者的引用不为0，无法通过引用计数算法进行回收
function foo() {
    const obj1 = {}
    const obj2 = {}
    obj1.name = obj2
    obj2.name = obj1

    return 'nothing'
}

foo()
```

# 标记清除算法
标记清除算法由**标记阶段**和**清除阶段**构成。在**标记阶段**会把所有的**活动对象都做上标记**，然后在**清除阶段**会把没有标记的对象，也就是**非活动对象回收**。
* 活动对象：能通过引用程序引用的对象就被称为活动对象。（可以直接或间接从全局变量空间中引出的对象）---> 可达
* 活动对象：不能通过程序引用的对象被称为非活动对象。---> 不可达

## 标记清除算法优点：
* 因为标记清除算法的标准是对象是否可达，所以能解决引用计数算法无法处理循环引用的问题。

## 标记清除算法缺点：
* 不能立即回收垃圾对象
* 清除阶段会把非活动对象回收再利用，由于被清除的非活动对象不是连续的且大小不一，导致空间碎片化严重，在重新分配使用时，不一定是适合的空间大小，每次都得遍历去找到合适的空间，分配速度慢。  
一篇来自掘金的文章：[垃圾回收算法|GC标记-清除算法](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6844903641791348744)

# 标记整理算法
标记整理算法是标记清除算法的增强，第一步标记阶段相同，第二阶段，在清除前，对空间进行整理，将非活动对象空间和空闲空间放到一起，最后清除回收非活动对象，这样过后，绝大部分空闲空间就是连续的，减少了空间碎片，在重新分配使用时效率大大提升。

# V8
题外话：V8是大理比较好喝的一种啤酒=。=！

* V8是一款主流的JavaScript执行引擎
* V8采用即时编译，速度快
* V8内存设限（32位800MB，64位1.5GB）（针对浏览器使用；依据其垃圾回收机制，内存太大，垃圾回收时间太长，会造成明显的卡顿）

## V8垃圾回收策略：
采用分代回收的思想，将内存分为新生代、老生代，针对不同的对象采用不用的算法。

## V8中常用GC算法：
* 分代回收
* 空间复制
* 标记清除
* 标记整理
* 标记增量

## V8回收新生代对象：（复制算法，空间换时间）
V8内存空间一分为二，小空间用于存储新生代对象（这个小空间大小为，64位32MB，32位16MB），新生代只存活时间较短的对象（如局部作用域的变量），新生代又分为两个相等的空间from space 与 to space，两个空间，同一时间内，只会有一个空间在工作( from space )，另一个在休息( to space )。

* 首先，V8 引擎中的垃圾回收器检测到 from space 空间快达到上限了，此时要进行一次垃圾回收了；
* 然后，从根部开始遍历，不可达对象将会被标记，并且复制未被标记的对象（活动对象），放到 to space 中；
* 最后，清除 from space 中的数据（非活动对象），同时将 from space 置为空闲状态，即变成 to space，相应的 to space 变成 from space，俗称翻转；

当经历一次 form => to 翻转之后，发现某些未被标记的对象居然还在，会直接扔到老生代里面去；或者在拷贝过程中to空间的使用率超过25%，就会把这次拷贝的数据扔到老生代去，保证翻转后form空间够用。

## V8回收老生代对象：
V8内存除新生代区域外大的部分（64位1.4GB，32位700MB），老生代对象是指存活时间比较长的对象（如全局变量，闭包产生变量数据等）。老生代区又可分为以下四个区域：
* old object space，这里的对象大部分是由新生代晋升而来；
* large object space，大对象存储区域，其他区域无法存储下的对象会被放在这里，基本是超过 1M 的对象，这种对象不会在新生代对象中分配，直接存放到这里；
* Map space，存储对象的映射关系的，其实就是隐藏类（隐藏类参考链接：V8引擎优化机制之隐藏类和内联缓存）；
* code space，简单点说，就是存放代码的地方，编译之后的代码；

回收过程主要采用标记清除、标记整理、增量标记算法：
* 首先，主要使用标记清除完成垃圾回收；
* 如果是新生代晋升的对象，且带到老生代后内存不足的情况下，就会触发标记整理进行空间优化；
* 最后采用增量标记进行效率优化；

增量标记：（摘自[掘金](https://juejin.cn/post/6844903619720904717)）

为了避免出现 JavaScript 应用逻辑与垃圾回收器看到的不一致的情况,垃圾回收的 3 种基本算法都需要将应用逻辑暂停下来，待执行完垃圾回收后再恢复执行应用逻辑,这种行为被称为“全停顿"，长时间的"全停顿"垃圾回收会让用户感受到明显的卡顿，带来体验的影响。以1.5 GB的垃圾回收堆内存为例,V8做一次小的垃圾回收需要50毫秒以上,做一次非增量式的垃圾回收甚至要1秒以上。这是垃圾回收中引起JavaScript线程暂停执行的时间,在 这样的时间花销下,应用的性能和响应能力都会直线下降。
为了降低全堆垃圾回收带来的停顿时间,V8先从标记阶段入手,将原本要一口气停顿完成的动作改为增量标记(incremental marking),也就是拆分为许多小“步进”,每做完一“步进” 就让 JavaScript 应用逻辑执行一小会儿,垃圾回收与应用逻辑交替执行直到标记阶段完成。

# 监控内存的几种方式
性能问题：
* 内存泄漏：内存使用持续升高；
* 内存膨胀：在多数设备上都存在性能问题；
* 频繁垃圾回收：通过内存变化图进行分析；

监控内存方式：
* 浏览器任务管理器；
* Timeline时序图记录（精确定位到内存与代码的相关性。）；
* 堆快照查找分离DOM（浏览器工具内存，堆快照，过滤“deta”查看分离DOM）；
* 判断是否存在频繁的垃圾回收（Timeline中频繁的上升下降；任务管理器中数据频繁的增加减少）；

# 浏览器任务管理器监控内存
浏览器中 **shift + esc** 调出任务管理器；JavaScript内存可明显看出代码层面产生的内存，能对整体的内存作出比较明显的观察判断；如果可达对象所占内存持续上升没有下降，则说明没有GC消耗或者内存上升大于GC消耗，存在性能隐患。
![图3](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/3.jpg)

# JavaScript代码性能测试
一下两个地址可对 JavaScript 代码进行性能测试：  
[https://jsperf.com](https://jsperf.com)  
[https://jsbench.me](https://jsbench.me)

# JavaScript代码优化
## 慎用全局变量：
全局变量定义在全局执行上下文，是所有作用域链的顶端；  
全局执行上下文一直存在于上下文执行栈，直到程序退出；  
如果某个局部作用域出现了同名变量则会屏蔽或污染全局；  
![图4](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/4.jpg)

## 缓存全局变量：
将使用中无法避免的全局变量缓存到局部。  
```javascript
// 直接使用全局变量
function getBtn0() {
    document.getElementById('btn1')
}

// 局部缓存全局变量 ==》 性能更优
function getBtn1() {
    let doc = document
    doc.getElementById('btn1')
}
```

## 通过原型新增方法：
在原型对象上新增实例对象需要的方法。
```javascript
function foo1() {
    this.fn = function () {
        console.log(123);
    }
}
let f1 = new foo1()

// 性能更优
function foo2() {}
foo2.prototype.fn = function () {
    console.log(123);
}
let f2 = new foo2()
```
![图5](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/5.jpg)

## 避开闭包陷阱：
闭包特点：  
* 外部具有指向内部的引用；
* 在“外”部作用域访问“内”部作用域的数据；
* 使用不当容易出现内存泄漏；
```javascript
function foo() {
    let el = document.getElementById('btn')
    el.onclick = function () {
        console.log(el.id);
    }
}
foo()

// el使用后，当btn被删除，el的引用也就不存在，不会造成内存泄漏
function foo() {
    let el = document.getElementById('btn')
    el.onclick = function () {
        console.log(el.id);
    }

    el = null
}
foo()
```

## 避免属性访问方法的使用：
* JavaScript不需要属性访问方法，所有属性都是外部可见的；
* 使用属性访问方法只会增加一层重定义，没有访问的限制能力；
```javascript
function Person1() {
    this.name = 'SeaChan'
    this.age = 18
    this.getAge = function () {
        return this.age
    }
}

const p1 = new Person1()
const age1 = p1.getAge()

// 性能更优
function Person2() {
    this.name = 'SeaChan'
    this.age = 18
}

const p2 = new Person2()
const age2 = p2.age
```
![图6](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/6.jpg)

## for循环优化：
集合长度定义变量，节省每次遍历的开销。
![图7](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/7.jpg)

## 采用最优循环方式：
forEach最快
![图8](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/8.jpg)

## 文档碎片优化节点添加：
![图9](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/9.jpg)

## 克隆优化节点操作：
![图10](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/10.jpg)

## 直接量替换Object操作：
![图11](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/11.jpg)

## 减少判断层次：（减少了逻辑嵌套）
![图12](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/12.jpg)

## 减少作用域链查找层次：
![图13](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/13.jpg)

## 减少数据读取次数：
![图14](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/14.jpg)

## 字面量与构造式：
构造式new的过程相较于字面量多了一次函数调用。
![图15](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/15.jpg)

## 减少循环体中的活动：
![图16](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/16.jpg)

![图17](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/17.jpg)

## 减少声明和语句数：
减少代码解析阶段的工作。
![图18](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/18.jpg)

![图19](https://raw.githubusercontent.com/SeaChan0117/SeaChan0117.github.io/master/source/_posts/javascript-optimize/19.jpg)

##采用事件委托：
```javascript
<ul id="ul">
    <li>SeaChan</li>
    <li>18</li>
    <li>JavaScript</li>
</ul>

// 单个li绑定事件
const foo1 = () => {
    const liArr = document.querySelectorAll('li')
    liArr.forEach(li => {
        li.onclick = function (e) {
            console.log(e.target.innerHTML);
        }
    })
}
foo1()

// 事件委托
const foo2 = () => {
    const ul = document.getElementById('ul')
    ul.addEventListener('click', function (e) {
        if (e.target.nodeName.toLowerCase() === 'li') {
            console.log(e.target.innerHTML);
        }
    }, true)
}
foo2()
```

# 总结：
Time is money！前端性能可优化的点还有很多，本文单指纯JavaScript层面的一些优化点，整体可以划分到解析、编译和执行几个阶段上。如果还有未涉及的部分，欢迎大家帮忙指出。
