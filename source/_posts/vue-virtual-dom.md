---
title: vue-virtual-dom
date: 2021-08-10 14:10:59
categories:
- Vue
tags:
- 虚拟DOM
---

学习 **vue** 源码前，需对 **虚拟Dom** 有一定的了解，本文将通过 **snabbdom** 库，对 **虚拟Dom** 进行学习，原因是 **Vue** 的 **虚拟Dom** 实现就是基于 **snabbdom** 改造的。

# 什么是虚拟Dom

**Virtual Dom**（虚拟Dom），是由普通的 **JS对象** 来描述 **Dom** 对象，个人理解就是 **真实Dom** 抽象成 **JS对象** 的最小化属性集，后续可根据它的相关属性生成一个对应的 **真实Dom**。由于 **真实Dom** 的属性繁多，而实际使用时大多是不需要的，所以 **虚拟Dom** 的从基础就决定了它有所占内存较小的优势。
<!--more-->
![打印一个真实Dom的所有属性](01.jpg)

![虚拟Dom示例，其就是一个JS对象](02.jpg)

**snabbdom** 等库对 **虚拟Dom** 的实现以及相关方法（重点是 **diff** 算法）能将 **虚拟Dom** 和 **真实Dom** 进行映射，并将更改后的状态同步到 **真实Dom**，避免了开发者手动操作 **Dom**。**虚拟Dom** 中存在多次操作，只会最终和 **真实Dom** 做对比，把修改的节点更新到 **真实Dom**，节省了操作次数，提高性能。

# 虚拟Dom的作用

- 维护视图和状态的关系
- 复杂视图情况下提升渲染性能
- 最大的好处是**跨平台**

> **浏览器平台渲染Dom**
> **服务端渲染SSR（Nuxt.js/Next.js）**
> **原生应用（Weex/React Native）**
> **小程序（mpvue/uni-app）**

# **snabbdom（虚拟Dom库）**

**Vue 2.x** 内部使用的 **虚拟Dom** 是基于 **snabbdom** 改造实现的，所以此处对 **snabbdom** 进行简单的学习，以便后续学习 **Vue** 时对 **虚拟Dom** 有一定了解。

- 通过模块可扩展
- 源码通过TS开发
- 最快的虚拟Dom之一

# snabbdom 简单示例上手

此处使用 **parcel** 进行打包，原因是代码结构简单，**parcel** 零配置，使用起来效率更高；

初始化项目后安装 **snabbdom** ；

创建 **js** 文件如下：

```js
import {init} from 'snabbdom/build/package/init'
import {h} from 'snabbdom/build/package/h'

const patch = init([])

// 第一个参数：标签+选择器
// 第二个参数：如果是字符串就是标签中的文本内容
let vNode = h('div#container.cls', 'Hello World')
let app = document.querySelector('#app')

// 第一个参数：旧的 vNode ，可以是 dom 元素
// 第二个参数：新的 vNode
// 返回新的 vNode
let oldVnode = patch(app, vNode)

vNode = h('div#id.class-test', 'Snabbdom')
patch(oldVnode, vNode)
```

在 **index.html** 中引入该 **js**，**parcel** 启动项目，`yarn parcel index.html --open`;

![虚拟Dom渲染到页面](03.jpg)

# 模块的使用

**snabbdom** 中的模块是用来扩展 **snabbdom** 的功能，类似于插件的机制；

- **snabbdom** 的核心库并不能处理 **Dom** 元素的属性、样式、事件等，可以通过注册 **snabbdom** 默认提供的模块（styleModule、eventListenersModule等）来实现；
- **snabbdom** 中的模块可以扩展 **snabbdom** 的功能；
- **snabbdom** 中模块的实现是通过注册全局的钩子函数来实现的；

# 官方提供的模块：

- **attributes**：设置 **vnode** 对应 **Dom** 属性（使用 **Dom** 的 **setAttribute** 实现）；
- **props**：设置 **vnode** 对应 **Dom** 属性（对象设置值的方式设置，a.b = 'xx'不会处理布尔类型的属性）；
- **dataset**：处理 H5 中 data-xxx 属性；
- **class**：切换类样式；
- **style**：设置行内样式；
- **eventlisteners**：注册和移除事件；

# 模块的使用步骤：

- 导入模块；
- **init()** 中注册模块；
- **h()** 函数的第二个参数处使用模块；

![](04.jpg)

![模块使用示例](05.jpg)

# diff 算法重点 updateChildren：

**snabbdom** 的核心方法入口 **patch**，意思为打补丁，即通过diff算法找到差异后打补丁，只针对差异去更新 **dom**，不是所有节点的删除重建。**diff** 方法作为核心是怎样查找差异节点的呢？

**snabbdom** 的 **diff** 算法只会在同层级节点进行比较，不会跨级比较。

如果同级别不相同，直接删除重新创建，同级节点只需要比较一次，较传统的比较大大减少了比较次数，性能也更优。

![diff算法同层级比较](06.jpg)

同级别节点比较时，会对新旧节点数组的开始和结束节点设置索引，遍历的过程中移动相应的索引，在对开始和结束节点的比较中有以下四种情况。

oldStartVnode/newStartVnode（旧开始节点/新开始节点）

> 新旧开始节点是sameVnode（key和sel相同），调用patchVnode()对比和更新节点；
> 把新旧节点开始索引往后移动 oldStartIdx++/newStartIdx++；

![](07.jpg)

oldEndVnode/newEndVnode（旧结束节点/新结束节点）

> 如果新旧开始节点不是sameVnode，就会从最后往前开始比较是否是sameVnode；
> 如果是sameVnode，就调用patchVnode()；
> 比较完成后会移动索至倒数第二个节点，比较判断是否是sameVnode，重复以上执行；

oldStartVnode/newEndVnode（旧开始节点/新结束节点）

> 首先通过sameVnode比较是否是相同节点；
> 是相同节点则调用patchVnode()对比和更新节点；
> 把oldStartVnode对应的Dom元素移动到右边，更新索引oldStartIdx++/newEndIdx--；

![](08.jpg)

oldEndVnode/newStartVnode（旧结束节点/新开始节点）

> 首先通过sameVnode比较是否是相同节点；
> 是相同节点则调用patchVnode()对比和更新节点；
> 把oldEndVnode对应的Dom元素移动到左边，更新索引oldEndIdx--/newStartIdx++；

![](09.jpg)

如果对比过程中都不是以上四种情况，开始和结束节点都不相同，就会去旧节点数组中依次查找是否有相同的新节点，首先遍历新的开始节点，在旧节点数组中查找是否有相同 **key** 值的节点，如果没有找到，说明此时的开始节点是一个新节点，要创建新的 **Dom** 元素，并把它插入到最前边的位置；如果新开始节点在旧节点数组中找到了具有相同 **key** 值的节点，并且判断新旧节点的sel属性是否相同，如果sel不相同，创建新的 **Dom** 元素，把它插入到最前边的位置，如果 **sel** 相同，说明是相同节点，此时这个旧节点会被赋值给 **elmToMove** 变量，再调用 **patchVnode** 对比和更新差异，再把 **elmToMove** 对应的 **Dom** 元素移动到最前边。

**注：是sameVnode时，会重用旧节点对应的dom元素，在patchVnode中对比新旧节点的差异，把差异更新到重用的dom元素上；**
