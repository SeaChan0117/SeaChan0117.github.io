---
title: 图片懒加载
date: 2021-12-10 09:41:00
categories: 
- 大前端
tags:
- 懒加载

cover: https://tse1-mm.cn.bing.net/th/id/R-C.3b5f9c1b7956978778678cb817eb5fa4?rik=E0rAxzwO6uDjSw&riu=http%3a%2f%2fwww.javanx.cn%2fwp-content%2fthemes%2flensnews2.2%2fimages%2fpost%2f20181101174122.gif&ehk=bsaah9nDJnimWy3yisy0z5fjhWaE7DmTIXcRf%2f4xHa8%3d&risl=&pid=ImgRaw&r=0
---

# 注
本文针对不理解懒加载实现或者没实践过懒加载实现的开发人员，老手可忽略=。=

# 前言
懒加载对于稍微有点开发经验的开发人员来说，多多少少都用过，至少听过，也都大概知道是为了用户体验优化性能而做的，说起来今天为什么突然想记录图片懒加载呢？

主要原因是，目前太多的插件或类库有现成的懒加载供开发者使用，越来越多的年轻开发者对懒加载的实现不是很在意，毕竟很多拿来就用了，正好自己最近在面试（还没被问过），顺带记录一下。

# 什么是图片懒加载
懒加载，即按需加载，需要的时候再加载，最常见的就是图片懒加载，树的懒加载，包括目前前端框架的一些组件或页面懒加载等；

图片懒加载，就是在用户打开一个有很多图片的页面时，初始化只加载用户可视区域的图片，当用户滚动时再将接下来 *要展示* 的图片加载，通过减少或者延迟请求次数，缓解浏览器压力，增强用户体验。

本文后续使用 *懒加载* 统一表示本文主题 *图片懒加载*

# 实现原理
## 没有懒加载
> 代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>No LazyLoad</title>
    <style>
        img {
            display: block;
            width: 300px;
            height: 200px;
        }
    </style>
</head>
<body>
<img src="imgs/1.jpg" alt="1">
<img src="imgs/2.jpg" alt="2">
<img src="imgs/3.jpg" alt="3">
<img src="imgs/4.jpg" alt="4">
<img src="imgs/5.jpg" alt="5">
<img src="imgs/6.jpg" alt="6">
<img src="imgs/7.jpg" alt="7">
<img src="imgs/8.jpg" alt="8">
<img src="imgs/9.jpg" alt="9">
</body>
</html>
```
> 页面结果

![全加载页面.png](全加载页面.png)

## 懒加载原理
懒加载实现原理我做了一张图示：
![图片懒加载.png](图片懒加载.png)

## 懒加载实现
懒加载要做以下几件事：
> 1. 初始化默认视口内的图片已加载；

> 2. 未加载的 `img` 默认通过一统一的图片占位，即一般使用一张 `loading` 状态的图，当加载过程慢时，给用户一种交互式的体验；

> 3. 未加载的 `img` 要预存真实图片的待加载路径，一般使用 `data-set`，即此处使用 `data-src` 预存加载路径；

> 3. 滚动时判断图片距离视窗顶部的距离，即上边原理图中的结论，当条件成立时，将 `img` 的 `src` 替换为 `data-src`；

> 4. `img` 标签通过替换过来的 `src` 去加载所需图片，懒加载完成；

上代码：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>LazyLoad</title>
    <style>
        img {
            display: block;
            width: 300px;
            height: 200px;
        }
    </style>
</head>
<body>
<img src="imgs/load.gif" data-src="imgs/1.jpg" alt="1">
<img src="imgs/load.gif" data-src="imgs/2.jpg" alt="2">
<img src="imgs/load.gif" data-src="imgs/3.jpg" alt="3">
<img src="imgs/load.gif" data-src="imgs/4.jpg" alt="4">
<img src="imgs/load.gif" data-src="imgs/5.jpg" alt="5">
<img src="imgs/load.gif" data-src="imgs/6.jpg" alt="6">
<img src="imgs/load.gif" data-src="imgs/7.jpg" alt="7">
<img src="imgs/load.gif" data-src="imgs/8.jpg" alt="8">
<img src="imgs/load.gif" data-src="imgs/9.jpg" alt="9">

<script>
    const imgs = document.getElementsByTagName('img')
    const len = imgs.length
    // 初始化默认视口内的图片已加载, lazyLoad 默认执行一遍
    lazyLoad()
    window.onscroll = lazyLoad

    function lazyLoad () {
        const H0 = document.documentElement.clientHeight
        const H1 = document.documentElement.scrollTop || document.body.scrollTop
        for (let i = 0; i < len; i++) {
            const img = imgs[i]
            if ((img.offsetTop < H0 + H1)) {
                if (img.getAttribute('src') === 'imgs/load.gif') {
                    img.src = img.getAttribute('data-src')
                }
            }
        }
    }
</script>
</body>
</html>

```

 懒加载结果：
 ![懒加载页面.png](懒加载页面.png)
 ![懒加载滚动.gif](懒加载滚动.gif)

可以看到，经过懒加载后，初始化只加载了 4 张图片，对比全加载时，减少了很多初始化时的请求，当浏滚动条滚动时，依次加载所需图片，一个简单的懒加载就实现了，是不是很简单呢。

## 优化
上述代码很明显可以看出两个问题：
* 滚动事件频繁触发，需要节流操作；
* 每次执行 `lazyLoad` 都会遍历所有的 `img`；

直接上代码：
```js
    const imgs = document.getElementsByTagName('img')
    const len = imgs.length

    // start 标记已加载过的图片索引
    let start = 0
    // 初始化默认视口内的图片已加载, lazyLoad 默认执行一遍
    lazyLoad()
    // throttle 节流
    window.onscroll = throttle(lazyLoad)

    function lazyLoad () {
        const H0 = document.documentElement.clientHeight
        const H1 = document.documentElement.scrollTop || document.body.scrollTop
        // start === len 后，lazyLoad 触发也不再进行加载操作，即所有图片加载完了
        for (let i = start; i < len; i++) {
            console.log(start)
            const img = imgs[i]
            if ((img.offsetTop < H0 + H1)) {
                if (img.getAttribute('src') === 'imgs/load.gif') {
                    img.src = img.getAttribute('data-src')
                    start++
                }
            }
        }
    }

    // 实现个简单的节流函数，也可使用第三方类库
    function throttle(fn) {
        return function (...args) {
            if (fn.flag) return
            fn.flag = true
            setTimeout(() => {
                fn(...args)
                Reflect.deleteProperty(fn, 'flag')
            }, 300)
        }
    }
```

通过标记加载过的图片索引和节流，针对上边的问题进行了优化，一个差不多的懒加载过程就这样完成了。

但是，还有一个问题，细心的同学应该也发现了，当我们滚动到最下边时，再刷新浏览器，你会发现所有的图片都加载了。

# IntersectionObserver
上边说到的问题我们使用 `IntersectionObserver API` 来解决。

## 什么是 IntersectionObserver 
MDN 上这样描述：

[IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver) 接口 (从属于Intersection Observer API) 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(viewport)交叉状态的方法。祖先元素与视窗(viewport)被称为根(root)。

简单来说，就是监听元素是否进入了视口；

具体用法可查阅相关资料，本文不做过多说明；

相关链接：  
[http://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html](http://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)

## 通过 IntersectionObserver API 改造优化懒加载

```js
    lazyLoad()

    function lazyLoad() {
        const imgs = document.getElementsByTagName('img')
        const len = imgs.length
        // 通过构造函数返回实例，实例中定义了进入视口的回调
        let lazyLoadObserver = new IntersectionObserver((entries, observer) => {
            entries.forEach((entry, index) => {
                let img = entry.target
                // intersectionRatio 相交率，默认是相对于浏览器视窗视口，大于 0 说明进入视口
                if (entry.intersectionRatio > 0) {
                    img.src = img.getAttribute('data-src')
                    // 当前图片加载完之后需要去掉监听
                    lazyLoadObserver.unobserve(img)
                }

            })
        })
        // 遍历 imgs，上边定义的 IntersectionObserver 实例去监听各 img
        for (let i = 0; i < len; i++) {
            lazyLoadObserver.observe(imgs[i])
        }
    }
```

实现结果：
![IntersectionObserver懒加载.png](IntersectionObserver懒加载.png)

可以看到，通过 `IntersectionObserver API` 实现的懒加载，在滚动条滚动到底部后刷新浏览器，初始化只加载最后几张图片，当向上滚动滚动条时，依次向上加载图片，
在图片加载后通过 `unobserve` 解除监听，同时解决了上边提到的三个问题，是不是很奈斯。

# Vue 自定义懒加载指令
既然实现了懒加载，那接下来就是套入常用的框架了，这里以 Vue 为例，将懒加载通过自定义指令方式实现。
## Vue 自定义指令
[https://cn.vuejs.org/v2/guide/custom-directive.html](https://cn.vuejs.org/v2/guide/custom-directive.html)
[https://juejin.cn/post/6844903537625808904](https://juejin.cn/post/6844903537625808904)

## 实现图片懒加载
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>LazyLoad</title>
    <style>
        img {
            display: block;
            width: 300px;
            height: 200px;
        }
    </style>
</head>
<body>
<div id="app">
    <template v-for="item in 9">
        <img v-lazyload="`imgs/${item}.jpg`" :alt="item" :key="`imgs/${item}.jpg`">
    </template>
</div>

<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<script>

    Vue.directive('lazyload', {
        bind: function (el, binding) {
            let lazyLoadObserver = new IntersectionObserver((entries, observer) => {
                entries.forEach((entry, index) => {
                    let img = entry.target
                    if (entry.intersectionRatio > 0) {
                        img.src = binding.value
                        lazyLoadObserver.unobserve(img)
                    }
                })
            })
            lazyLoadObserver.observe(el)
        }
    })
    new Vue({
        el: '#app'
    })

</script>
</body>
</html>

```

以上结果验证懒加载 OK，一个简单的 Vue 懒加载自定义指令就完成啦。

# 后续
## 2021-12-14 面试
2021-12-14 面试必要工业科技的前端岗位，面试官针对我这篇文章问了一些我对懒加载实现的思路，并且在这个基础上做了一些延展出去的场景和问题
### 问题一
页面图片很多，比如一千张，用户直接拖动滚动条到底部，这时上边未显示的图片加载了吗？为什么？怎么处理？  
答：
> 因为这篇文章一开始没打算把懒加载做得那么完善，这个问题确实没想过，上边的图确实会加载（思考了一下），所以也确实还有性能问题，因为不管是一开始实现的监听滚动事件还是 IntersectionObserver API 的实现方法，滚动过程中都上边未在视口中的图都经历了懒加载的逻辑，其实也是懒加载，但是这个场景下，用户没打算看上边那些，还是会造成性能问题或者是资源浪费的问题，有违兰懒加载的初衷；
> 面试时针对这个问题我想的是滚动结束的时候再进懒加载的逻辑，并且懒加载的逻辑里边还要判断图片是否在视口内，包括一开始实现的图片顶部到视口的距离小于视口加滚动距离，还需要加上图片底部到视口底部的距离要小于视口高度，这样就能完善该问题；

#### 实现
```js
const imgs = [...document.getElementsByTagName('img')]
    // 初始化默认视口内的图片已加载, lazyLoad 默认执行一遍
    lazyLoad()
    // debounce 防抖，使用防抖函数在滚动停止间歇大于等于 delay 时，执行（即面试官问题中滚动停止到当前视口时，才执行）
    window.onscroll = debounce(lazyLoad)

    function lazyLoad () {
        const H0 = document.documentElement.clientHeight
        const H1 = document.documentElement.scrollTop || document.body.scrollTop
        for (let i = 0; i < imgs.length; i++) {
            const img = imgs[i]
            if ((img.offsetTop < H0 + H1) && (img.offsetTop + img.offsetHeight > H1)) {
                if (img.getAttribute('src') === 'imgs/load.gif') {
                    img.src = img.getAttribute('data-src')
                    // 从待加载图片集合中移除，全部移除后，后续不进此循环，减少不必要操作，此时 i-1 保证下一个循环的图片正确
                    imgs.splice(i, 1)
                    i--
                }
            }
        }
    }

    // 防抖函数，滚动停止间歇大于等于 delay 时，执行
    function debounce(fn, delay = 300) {
     return function (...args) {
      clearTimeout(fn.timer)
      fn.timer = setTimeout(() => {
       fn(...args)
      }, delay)
     }
    }
```

![debounce.gif](debounce.gif)

通过 debounce 修改后，得到上图结果，拖动结束时才触发懒加载，减少直接下拉太多造成的不必要加载，同时也保证了初始化默认在非顶部时刷新只加载当前视口内图片；每加载一张图片后通过 splice 将其从待加载图片集合中移除，保证后续的重复加载，减少额外的性能消耗。

### 问题二
图片加载失败的时候，需要显示一张默认裂开的图片，怎么处理？  
答：
> 拿到待加载图片 src 时不要直接替换页面上 img 标签的 src，通过 Image 实例设置 src 的 onerror 回调判断获取失败，并且设置 img 标签的 src 是某一默认的裂开图片。

#### 实现
```js
    const imgs = [...document.getElementsByTagName('img')]
    // 初始化默认视口内的图片已加载, lazyLoad 默认执行一遍
    lazyLoad()
    // debounce 防抖
    window.onscroll = debounce(lazyLoad)
    function lazyLoad () {
        const H0 = document.documentElement.clientHeight
        const H1 = document.documentElement.scrollTop || document.body.scrollTop
        for (let i = 0; i < imgs.length; i++) {
            const img = imgs[i]
            if ((img.offsetTop < H0 + H1) && (img.offsetTop + img.offsetHeight > H1)) {
                if (img.getAttribute('src') === 'imgs/load.gif') {
                    // img.src = img.getAttribute('data-src')
                    loadImage(img, img.getAttribute('data-src'))
                    // 从待加载图片集合中移除，全部移除后，后续不进此循环，减少不必要操作，此时 i-1 保证下一个循环的图片正确
                    imgs.splice(i, 1)
                    i--
                }
            }
        }
    }

    // 加载图片方法，失败时默认展示 errPath
    function loadImage(imgEl, src, errPath = 'imgs/error.jpg') {
        let img = new Image()
        img.onload = function (e) {
            imgEl.src = src
        }
        img.onerror = function (e) {
            imgEl.src = errPath
        }
        img.src = src
    }
    
    // 防抖函数，滚动停止间歇大于等于 delay 时，执行
    function debounce(fn, delay = 300) {
        return function (...args) {
            clearTimeout(fn.timer)
            fn.timer = setTimeout(() => {
                fn(...args)
            }, delay)
        }
    }
```
![加载失败展示默认一张裂开图片.png](加载失败展示默认一张裂开图片.png)


经过实践，证明了面试时的问题应该 OK。

### 问题四
基于上一个问题，假如加载失败了让它重新再加载试一次呢，如果第二次成功就用成功的，否则就展示裂开？  
答：
> 加载图片方法中失败了我再调一次加载方法，甚至可以把重试次数作为参数，失败一次重试次数减一，再调加载方法就行。

#### 实现
```js
    const imgs = [...document.getElementsByTagName('img')]
    // 初始化默认视口内的图片已加载, lazyLoad 默认执行一遍
    lazyLoad()
    // debounce 防抖
    window.onscroll = debounce(lazyLoad)
    function lazyLoad () {
        const H0 = document.documentElement.clientHeight
        const H1 = document.documentElement.scrollTop || document.body.scrollTop
        for (let i = 0; i < imgs.length; i++) {
            const img = imgs[i]
            if ((img.offsetTop < H0 + H1) && (img.offsetTop + img.offsetHeight > H1)) {
                if (img.getAttribute('src') === 'imgs/load.gif') {
                    // img.src = img.getAttribute('data-src')
                    loadImage(img, img.getAttribute('data-src'))
                    // 从待加载图片集合中移除，全部移除后，后续不进此循环，减少不必要操作，此时 i-1 保证下一个循环的图片正确
                    imgs.splice(i, 1)
                    i--
                }
            }
        }
    }

    // 加载图片方法，失败时默认展示 errPath， attempt 加载错误后重试次数
    function loadImage(imgEl, src, attempt = 3, errPath = 'imgs/error.jpg') {
        const img = new Image()
        img.onload = function (e) {
            imgEl.src = src
        }
        img.onerror = function (e) {
            if (attempt === 0) {
                return imgEl.src = errPath
            }
           loadImage(imgEl, src, --attempt, errPath)
        }
        img.src = src
    }
    
    // 防抖函数，滚动停止间歇大于等于 delay 时，执行
    function debounce(fn, delay = 300) {
        return function (...args) {
            clearTimeout(fn.timer)
            fn.timer = setTimeout(() => {
                fn(...args)
            }, delay)
        }
    }
```

![加载失败重试.png](加载失败重试.png)

添加 attempt 参数和相关处理后，默认 attempt 重试 3 次，可以看出，图片找不到时重试了 3 次，加上一开始的一次，总的对该路径请求了 4 次。

bingo，我们的懒加载又强大了！

### 面试总结
这次换工作唯一一个去现场面的，本来一开始计划都是线上面的，虽然自己对“必要”没有很了解，但莫名之中对它有很大的期待吧，也还有就是知道当天都面完，自己不用跑很多次。

面试前了解到是两个技术面，顺利的话加 HR 三面。实际是四面 T T，前端面试官 + 前端负责人 + 技术总监 + HR。

三个技术面都没有太着重的揪着问一些技术理论上的点，就像上边懒加载这个问题一样，很多都是具体的场景延伸出的问题，让你给解决方法或者一些思路，能看出面试官们比较老道，不是那种背一堆理论知识就能简单应付的，
他们很看重你解决问题的能力和学习能力。整个面试过程了解到他们团队的技术范围，也能感受到这样一个团队能对个人的成长是非常大的。HR 面的话主要了解了一些我的工作经历和现在的找工作情况，也和我讲了一些公司的业务及相关情况，包括一些加班状态啥的。

对这次面试的整体感觉挺好，面试官都挺不错的，整个技术团队是比较有吸引力的。

看后续吧，加油学习去咯。
