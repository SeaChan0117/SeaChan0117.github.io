---
title: Vue3 响应式原理
date: 2021-07-18 17:04:01
categories: Vue3
tags: Vue3
---

# Vue3 响应式原理回顾
* Proxy 对象实现属性监听
* 多层属性嵌套时，在访问属性过程中处理下一级属性
* 默认监听动态添加的属性
* 默认监听属性的删除操作
* 默认监听数组索引和 length 属性
* 可以作为单独的模块使用
<!--more-->
# Proxy 对象

**Proxy** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

## 严格模式

set 和 deleteProperty 中需要返回布尔类型的值，在严格模式下，如果返回 false 的话会出现 Type Error 的异常；

```javascript
const target = {
    foo: 'foo',
    bar: 'bar'
}
// Reflect.getPrototypeOf()
// Object.getPrototypeOf()
const proxy = new Proxy(target, {
    get (target, key, receiver) {
        // return target[key]
        return Reflect.get(target, key, receiver)
    },
    set (target, key, value, receiver) {
        // target[key] = value
        return Reflect.set(target, key, value, receiver)
    },
    deleteProperty (target, key) {
        // delete target[key]
        return Reflect.deleteProperty(target, key)
    }
})

proxy.foo = 'foo123'
delete proxy.foo
```

## receiver

* Proxy 中 receiver：Proxy 或者继承 Proxy 的对象
* Reflect 中 receiver： 如果 target 对象中设置了 getter，getter 中的 this 指向 receiver

```javascript
const obj = {
    get foo() {
        console.log(this)
        return this.bar
    }
}
const proxy = new Proxy(obj, {
    get (target, key) {
        if (key === 'bar') {
            return 'value - bar'
        }
        return Reflect.get(target, key)
    }
})
console.log(proxy.foo)
```

![不设置receiver](2021-07-18_174107.png)

```javascript
const obj = {
    get foo() {
        console.log(this)
        return this.bar
    }
}
const proxy = new Proxy(obj, {
    get (target, key, receiver) {
        if (key === 'bar') {
            return 'value - bar'
        }
        return Reflect.get(target, key, receiver)
    }
})
console.log(proxy.foo)
```

![设置receiver](2021-07-18_174344.png)
