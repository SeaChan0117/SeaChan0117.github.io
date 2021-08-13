---
title: redux
date: 2021-08-11 14:44:39
categories:
- React
- Redux
tags:
- React
- Redux
---

# Redux 介绍

JavaScript 状态容器，提供可预测化的状态管理；
<!--more-->
![redux 工作流程](redux-workflow.png)

## 基本概念

### Store

Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。

Redux 提供`createStore`这个函数，用来生成 Store。

### State

`Store`对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。

当前时刻的 State，可以通过`store.getState()`拿到。

Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。

### Action

State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。

Action 是一个对象。其中的`type`属性是必须的，表示 Action 的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/acdlite/flux-standard-action)可以参考。

可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。

### Action Creator

View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。

### store.dispatch()

`store.dispatch()`是 View 发出 Action 的唯一方法。

### Reducer

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。

Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

> Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。

### store.subscribe()

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

显然，只要把 View 的更新函数（对于 React 项目，就是组件的`render`方法或`setState`方法）放入`listen`，就会实现 View 的自动渲染。

`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。

## 使用方法简单案例

直接上代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<button id="plus">+</button>
<span id="count">0</span>
<button id="minus">-</button>

<script src="./redux.js"></script>
<script>
    // 3.存储默认状态
    const initialState = {count: 0}

    // 2.创建 reducer 函数
    function reducer(state = initialState, action) {
        switch (action.type) {
            case 'increment':
                return {count: state.count + 1}
            case 'decrement':
                return {count: state.count - 1}
            default:
                return state
        }
    }

    // 1.创建store
    const store = Redux.createStore(reducer)
    // 4.定义 action
    const increment = {type: 'increment'}
    const decrement = {type: 'decrement'}

    // 5.给按钮添加点击事件
    document.getElementById('plus').onclick = () => {
        // 6.触发 action
        store.dispatch(increment)
    }
    document.getElementById('minus').onclick = () => {
        // 6.触发 action
        store.dispatch(decrement)
    }

    // 7.订阅 store
    store.subscribe(() => {
        document.getElementById('count').innerText = store.getState().count
    })
</script>
</body>
</html>
```

# React + Redux

## 单独使用 React 的问题

在 React 中组件通信的数据流是单向的，顶层组件可以通过 props 属性向下层组件传递数据，而下层组件不能向上层组件传递数据，要实现下层组件修改数据，需要上层组件传递修改数据的方法到下层组件，当项目越来越大的时候，组件之间的传递数据变得越来越困难。

## 加入 Redux 的好处

使用 Redux 管理数据，由于 Store 独立于组件，使得数据管理独立于组件，解决了组件之间传递数据困难的问题。

# 相关链接

[https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html](https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)

未完待续。。。
