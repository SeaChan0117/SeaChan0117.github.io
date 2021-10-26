---
title: Vue3 响应式原理手写实现
date: 2021-07-18 17:04:01
categories: Vue3
tags: Vue3
cover: https://res.cloudinary.com/dgvkvirqh/image/upload/v1603096497/1_xg4ZkSYAKYoJ5TV_vrEVug_ybnh6m.png
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

# 手写 reactive 函数

## reactive 

* 接收一个参数，判断参数是否是对象
* 创建拦截器对象 handler，设置 get / set / deleteProperty
* 返回 Proxy 对象

## 手写实现

reactive 文件夹下新建 index.js 文件，内容如下：

```javascript
const isObject = val => val !== null && typeof val === 'object'
const convert = target => isObject(target) ? reactive(target) : target
const hasOwnProperty = Object.prototype.hasOwnProperty
const hasOwn = (target, key) => hasOwnProperty.call(target, key)

export function reactive(target) {
    if (!isObject(target)) return
    const handler = {
        get(target, key, receiver) {
            // 收集依赖
            console.log('get', key)
            const result = Reflect.get(target, key, receiver)
            return convert(result)
        },
        set(target, key, value, receiver) {
            const oldValue = Reflect.get(target, key, receiver)
            let result = true
            if (oldValue !== value) {
                result = Reflect.set(target, key, value, receiver)
                // 触发更新
                console.log('set', key, value)
            }
            return result
        },
        deleteProperty(target, key) {
            const hasKey = hasOwn(target, key)
            const result = Reflect.deleteProperty(target, key)
            if (hasKey && result) {
                // 触发更新
                console.log('delete', key)
            }
            return result
        }
    }
    return new Proxy(target, handler)
}
```

# 响应式系统原理 effect && track

在上述 index.js 中继续编写代码：

## effect 手写实现

```javascript
let activeEffect = null
export function effect(callback) {
    activeEffect = callback
    callback() // 访问响应式对象属性，去收集依赖
    activeEffect = null
}

```

## track 手写实现

```javascript
let targetMap = new WeakMap()
export function track(target, key) {
    if (!activeEffect) return
    let depsMap = targetMap.get(target)
    if (!depsMap) {
        targetMap.set(target, (depsMap = new Map()))
    }

    let dep = depsMap.get(key)
    if (!dep) {
        depsMap.set(key, (dep = new Set()))
    }
    dep.add(activeEffect)
}
```

## reactive 中收集依赖调用 track

```javascript
get(target, key, receiver) {
    // 收集依赖
    console.log('get', key)
    track(target, key)
    const result = Reflect.get(target, key, receiver)
    return convert(result)
}
```

# 响应式系统原理 trigger

收集依赖结束后，在值修改时，需要触发更新，trigger 函数的作用就在于此

## trigger 手写实现

```javascript
export function trigger(target, key) {
    const depsMap = targetMap.get(target)
    if (!depsMap) return
    const dep = depsMap.get(key)
    if (dep) {
        dep.forEach(effect => {
            effect()
        })
    }
}
```

## reactive 中触发更新调用 trigger

```javascript
set(target, key, value, receiver) {
    const oldValue = Reflect.get(target, key, receiver)
    let result = true
    if (oldValue !== value) {
        result = Reflect.set(target, key, value, receiver)
        // 触发更新
        trigger(target, key)
    }
    return result
},
deleteProperty(target, key) {
    const hasKey = hasOwn(target, key)
    const result = Reflect.deleteProperty(target, key)
    if (hasKey && result) {
        // 触发更新
        trigger(target, key)
    }
    return result
}
```

# 响应式系统原理 ref

ref 接收一个原始值或者对象

## ref 手写实现

```javascript
export function ref(raw) {
    // 判断 raw 是否是 ref 创建的对象，如果是的话直接返回
    if (isObject(raw) && raw.__v_isRef) return
    let value = convert(raw)
    const r = {
        __v_isRef: true,
        get value() {
            track(r, 'value')
            return value
        },
        set value(newValue) {
            if (newValue !== value) {
                raw = newValue
                value = convert(raw)
                trigger(r, 'value')
            }
        }
    }
    return r
}
```

## reactive VS ref

* ref 可以把基本数据类型的数据，转换成响应式对象
* ref 返回的对象，重新赋值成对象也是响应式的
* reactive 返回的对象，重新赋值后丢失响应式
* reactive 返回的对象不可以解构

# 响应式系统原理 toRefs

接收一个对象作为参数，它会遍历对象身上的所有属性，然后挨个设置成响应式数据

## toRefs 手写实现

```javascript
export function toRefs(proxy) {
    const ret = proxy instanceof Array ? new Array(proxy.length) : {}
    for(const key in proxy) {
        ret[key] = toProxyRef(proxy, key)
    }
    return ret
}

function toProxyRef(proxy, key) {
    const r = {
        __v_isRef: true,
        get value() {
            return proxy[key]
        },
        set value(newValue) {
            proxy[key] = newValue
        }
    }
    return r
}
```

# 响应式系统原理 computed

## computed 手写实现

```javascript
export function computed(getter) {
    const res = ref()
    effect(() => (res.value = getter()))
    return res
}
```

