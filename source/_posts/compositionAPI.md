---
title: Composition API
date: 2021-07-17 17:36:18
categories: Vue3
tags:Vue3
---

# Composition API

## Composition API

Composition API 是 vue3 中新增的 API，使用传统的option配置方法写组件的时候问题，随着业务复杂度越来越高，代码量会不断的加大；由于相关业务的代码需要遵循option的配置写到特定的区域，导致后续维护非常的复杂，同时代码可复用性不高，而composition-api就是为了解决这个问题而生的。

Composition API 提供了几个函数：

* reactive

  相当于当前的 Vue.observable () API，经过reactive处理后的函数能变成响应式的数据，类似于option api里面的data属性的值

* watchEffect

  * watch 函数的简化版本，也用来监视数据的变化
  * 接收一个函数作为参数，监听函数内响应式数据的变化

* computed

  计算属性 API 

* ref

  接受一个内部值并返回一个响应式且可变的 ref 对象。ref 对象具有指向内部值的单个 property `.value`，可以把基本类型的数据包装成响应式对象

* toRefs

  把对象的每个属性都转化为响应式数据，就可以解构 toRefs 返回的对象，解构的属性也是响应式的，这个属性的值也是一个对象，存在一个 value 属性，在模板中取值可以省略 value，代码中不能省略

* 生命周期 hooks

  方法与option api基本一致，在对应的方法前加上 ```on```



