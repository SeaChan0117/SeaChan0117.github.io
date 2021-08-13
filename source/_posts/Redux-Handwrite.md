---
title: Redux 源码手写实现
date: 2021-08-13 16:10:35
categories:
- React
- Redux
tags:
- React
- Redux
---

![redux 工作流程](redux-workflow.png)
<!--more-->
## 核心逻辑

```javascript
/**
 * createStore(reducer, preloadedState, enhancer)
 * getState、dispatch、subscribe
 */
function createStore(reducer, preloadedState) {
    // store 对象存储的状态
    let currentState = preloadedState;
    // 存放订阅者函数
    let currentListeners = []

    // 获取状态
    function getState() {
        return currentState
    }

    // 触发 action
    function dispatch(action) {
        currentState = reducer(currentState, action)
        // 循环数组，调用订阅者
        for (let i = 0; i < currentListeners.length; i++) {
            const listener = currentListeners[i]
            // 调用订阅者
            listener()
        }
    }

    // 订阅状态
    function subscribe(listener) {
        currentListeners.push(listener)
    }

    return {
        getState,
        dispatch,
        subscribe
    }
}
```

计数器案例测试

```html
<button id="btn1">+1</button>
<span id="count">0</span>
<button id="btn2">-1</button>
<script src="./myRedux.js"></script>
<script>
    function reducer(state, action) {
        switch (action.type) {
            case 'increment':
                return state + 1
            case 'decrement':
                return state - 1
            default:
                return state
        }
    }

    const store = createStore(reducer, 0)
    console.log(store.getState())

    document.getElementById('btn1').onclick = () => {
        store.dispatch({type: 'increment'})
    }
    document.getElementById('btn2').onclick = () => {
        store.dispatch({type: 'decrement'})
    }
    store.subscribe(() => {
        document.getElementById('count').innerText = store.getState()
    })
</script>
```

## 参数类型约束

* reducer 必须是函数；
* action 必须是对象，并且必须有 type 属性；

```javascript
/**
 * createStore(reducer, preloadedState, enhancer)
 * getState、dispatch、subscribe
 */
function createStore(reducer, preloadedState) {
    // 约束 reducer 参数类型
    if (typeof reducer !== 'function') throw new TypeError('reducer must be a function!')
    // store 对象存储的状态
    let currentState = preloadedState;
    // 存放订阅者函数
    let currentListeners = []

    // 获取状态
    function getState() {
        return currentState
    }

    // 触发 action
    function dispatch(action) {
        // 判断 action 是否是对象
        if (!isPlainObject(action)) throw new TypeError('action must be an object')
        // 判断 action 中是否有 type 属性
        if (typeof action.type === 'undefined') throw new Error('action must have type character')
        currentState = reducer(currentState, action)
        // 循环数组，调用订阅者
        for (let i = 0; i < currentListeners.length; i++) {
            const listener = currentListeners[i]
            // 调用订阅者
            listener()
        }
    }

    // 订阅状态
    function subscribe(listener) {
        currentListeners.push(listener)
    }

    return {
        getState,
        dispatch,
        subscribe
    }
}

// 判断 obj 是否是对象
function isPlainObject(obj) {
    if (typeof obj !== 'object' || obj === null) return false

    let proto = obj
    while (Object.getPrototypeOf(proto) !== null) {
        proto = Object.getPrototypeOf(proto)
    }

    return Object.getPrototypeOf(obj) === proto
}
```

## Enhancer

enhancer 的作用是，让 createStore 方法的调用者对返回的 store 进行功能上的增强；

```javascript
/**
 * createStore(reducer, preloadedState, enhancer)
 * getState、dispatch、subscribe
 */
function createStore(reducer, preloadedState, enhancer) {
    // 约束 reducer 参数类型
    if (typeof reducer !== 'function') throw new TypeError('reducer must be a function!')

    // 判断 enhancer 是否有传
    if (typeof enhancer !== 'undefined') {
        // 判断 enhancer 是否是函数
        if (typeof enhancer !== 'function') throw new TypeError('enhancer must be a function!')

        return enhancer(createStore)(reducer, preloadedState)
    }

    // store 对象存储的状态
    let currentState = preloadedState;
    // 存放订阅者函数
    let currentListeners = []

    // 获取状态
    function getState() {
        return currentState
    }

    // 触发 action
    function dispatch(action) {
        // 判断 action 是否是对象
        if (!isPlainObject(action)) throw new TypeError('action must be an object')
        // 判断 action 中是否有 type 属性
        if (typeof action.type === 'undefined') throw new Error('action must have type character')
        currentState = reducer(currentState, action)
        // 循环数组，调用订阅者
        for (let i = 0; i < currentListeners.length; i++) {
            const listener = currentListeners[i]
            // 调用订阅者
            listener()
        }
    }

    // 订阅状态
    function subscribe(listener) {
        currentListeners.push(listener)
    }

    return {
        getState,
        dispatch,
        subscribe
    }
}

// 判断 obj 是否是对象
function isPlainObject(obj) {
    if (typeof obj !== 'object' || obj === null) return false

    let proto = obj
    while (Object.getPrototypeOf(proto) !== null) {
        proto = Object.getPrototypeOf(proto)
    }

    return Object.getPrototypeOf(obj) === proto
}
```

计数器案例测试增强后异步操作

```html
<button id="btn1">+1</button>
<span id="count">0</span>
<button id="btn2">-1</button>
<script src="./myRedux.js"></script>
<script>
    function enhancer(createStore) {

        return function (reducer, preloadedState) {
            let store = createStore(reducer, preloadedState)
            console.log(store)

            let dispatch = store.dispatch

            function _dispatch(action) {
                if (typeof action === 'function'){
                    return action(dispatch)
                }
                dispatch(action)
            }

            return {
                ...store,
                dispatch: _dispatch
            }
        }
    }

    function reducer(state, action) {
        switch (action.type) {
            case 'increment':
                return state + 1
            case 'decrement':
                return state - 1
            default:
                return state
        }
    }

    const store = createStore(reducer, 0, enhancer)
    console.log(store.getState())

    document.getElementById('btn1').onclick = () => {
        // store.dispatch({type: 'increment'})
        store.dispatch(dispatch => {
            setTimeout(() => {
                dispatch({type: 'increment'})
            }, 1000)
        })
    }
    document.getElementById('btn2').onclick = () => {
        store.dispatch({type: 'decrement'})
    }

    store.subscribe(() => {
        document.getElementById('count').innerText = store.getState()
    })
</script>
```

## applyMiddleware

将 enhancer 升级为 applyMiddleware 加强 dispatch 的功能；

新建两个中间件函数 logger.js 和 thunk.js ；

```javascript
function logger(store) {
    return function (next) {
        return function (action) {
            console.log('logger')
            next(action)
        }
    }
}
```

```javascript
function thunk(store) {
    return function (next) {
        return function (action) {
            console.log('thunk')
            next(action)
        }
    }
}
```

myRedux.js 修改，新增 applyMiddleware 函数

```javascript
/**
 * createStore(reducer, preloadedState, enhancer)
 * getState、dispatch、subscribe
 */
function createStore(reducer, preloadedState, enhancer) {
    // 约束 reducer 参数类型
    if (typeof reducer !== 'function') throw new TypeError('reducer must be a function!')

    // 判断 enhancer 是否有传
    if (typeof enhancer !== 'undefined') {
        // 判断 enhancer 是否是函数
        if (typeof enhancer !== 'function') throw new TypeError('enhancer must be a function!')

        return enhancer(createStore)(reducer, preloadedState)
    }

    // store 对象存储的状态
    let currentState = preloadedState;
    // 存放订阅者函数
    let currentListeners = []

    // 获取状态
    function getState() {
        return currentState
    }

    // 触发 action
    function dispatch(action) {
        // 判断 action 是否是对象
        if (!isPlainObject(action)) throw new TypeError('action must be an object')
        // 判断 action 中是否有 type 属性
        if (typeof action.type === 'undefined') throw new Error('action must have type character')
        currentState = reducer(currentState, action)
        // 循环数组，调用订阅者
        for (let i = 0; i < currentListeners.length; i++) {
            const listener = currentListeners[i]
            // 调用订阅者
            listener()
        }
    }

    // 订阅状态
    function subscribe(listener) {
        currentListeners.push(listener)
    }

    return {
        getState,
        dispatch,
        subscribe
    }
}

// 判断 obj 是否是对象
function isPlainObject(obj) {
    if (typeof obj !== 'object' || obj === null) return false

    let proto = obj
    while (Object.getPrototypeOf(proto) !== null) {
        proto = Object.getPrototypeOf(proto)
    }

    return Object.getPrototypeOf(obj) === proto
}

function applyMiddleware(...middlewares) {
    return function (createStore) {
        return function (reducer, preloadedState) {
            // 创建 store
            let  store = createStore(reducer, preloadedState)
            // 阉割版的 store
            let middleWareAPI = {
                getState: store.getState,
                dispatch: store.dispatch
            }

            // 调用中间件的第一层函数，传递阉割版的 store
            let chain = middlewares.map(middleWare => middleWare(middleWareAPI))
            let dispatch = compose(...chain)(store.dispatch)
            return {
                ...store,
                dispatch
            }
        }
    }
}

function compose () {
    let funcs = [...arguments]
    return function (dispatch) {
        for (let i = funcs.length - 1; i >= 0; i--) {
            dispatch = funcs[i](dispatch)
        }
        return dispatch
    }
}
```

案例测试：

```html
<button id="btn1">+1</button>
<span id="count">0</span>
<button id="btn2">-1</button>
<script src="./myRedux.js"></script>
<script src="./middlewares/logger.js"></script>
<script src="./middlewares/thunk.js"></script>
<script>
    function enhancer(createStore) {

        return function (reducer, preloadedState) {
            let store = createStore(reducer, preloadedState)
            console.log(store)

            let dispatch = store.dispatch

            function _dispatch(action) {
                if (typeof action === 'function'){
                    return action(dispatch)
                }
                dispatch(action)
            }

            return {
                ...store,
                dispatch: _dispatch
            }
        }
    }

    function reducer(state, action) {
        switch (action.type) {
            case 'increment':
                return state + 1
            case 'decrement':
                return state - 1
            default:
                return state
        }
    }

    const store = createStore(reducer, 0, applyMiddleware(logger, thunk))
    console.log(store.getState())

    document.getElementById('btn1').onclick = () => {
        // logger => thunk => reducer
        store.dispatch({type: 'increment'})
        // store.dispatch(dispatch => {
        //     setTimeout(() => {
        //         dispatch({type: 'increment'})
        //     }, 1000)
        // })
    }
    document.getElementById('btn2').onclick = () => {
        store.dispatch({type: 'decrement'})
    }

    store.subscribe(() => {
        document.getElementById('count').innerText = store.getState()
    })
</script>
```

注册中间件后，每次执行 action 前，都会按顺序先执行中间件；

## bindActionCreators

bindActionCreators 的作用是将 actionCreators 函数转换为可以出发 action 的函数；

myRedux.js

```javascript
...
function bindActionCreators(actionCreators, dispatch){
    let boundActionCreators = {}
    for(let key in actionCreators) {
        // dispatch(actionCreators[key])
        boundActionCreators[key] = function (){
            dispatch(actionCreators[key]())
        }
    }

    return boundActionCreators
}
```

测试

```html
<span id="count">0</span>
<button id="btn2">-1</button>
<script src="./myRedux.js"></script>
<script src="./middlewares/logger.js"></script>
<script src="./middlewares/thunk.js"></script>
<script>
    function enhancer(createStore) {

        return function (reducer, preloadedState) {
            let store = createStore(reducer, preloadedState)
            console.log(store)

            let dispatch = store.dispatch

            function _dispatch(action) {
                if (typeof action === 'function'){
                    return action(dispatch)
                }
                dispatch(action)
            }

            return {
                ...store,
                dispatch: _dispatch
            }
        }
    }

    function reducer(state, action) {
        switch (action.type) {
            case 'increment':
                return state + 1
            case 'decrement':
                return state - 1
            default:
                return state
        }
    }

    const store = createStore(reducer, 0, applyMiddleware(logger, thunk))
    console.log(store.getState())

    function increment() {
        return {type: 'increment'}
    }

    function decrement() {
        return {type: 'decrement'}
    }

    let actions = bindActionCreators({increment, decrement}, store.dispatch)

    document.getElementById('btn1').onclick = () => {
        // logger => thunk => reducer
        // store.dispatch({type: 'increment'})
        // store.dispatch(dispatch => {
        //     setTimeout(() => {
        //         dispatch({type: 'increment'})
        //     }, 1000)
        // })

        actions.increment()
    }
    document.getElementById('btn2').onclick = () => {
        // store.dispatch({type: 'decrement'})
        actions.decrement()
    }

    store.subscribe(() => {
        document.getElementById('count').innerText = store.getState()
    })
</script>
```

## combineReducers

combineReducers 将拆分的 reducer 合并为一个；

myRedux.js

```javascript
...
function combineReducers(reducers) {
    // 检查 reducer 类型
    let reducerKeys = Object.keys(reducers)
    for (let i = 0; i < reducerKeys.length; i++) {
        let key = reducerKeys[i]
        if (typeof reducers[key] !== 'function') throw new TypeError('reducer must be a function!')
    }
    // 调用一个个小的 reducer，将每个小的 reducer 返回的状态存储在一个大的对象中
    return function (state, action) {
        let nextState = {}

        for (let i = 0; i < reducerKeys.length; i++) {
            let key = reducerKeys[i]
            let reducer = reducers[key]
            let prevState = state[key]
            nextState[key] = reducer(prevState, action)
        }

        return nextState
    }
}
```

测试，注意在 subscribe 时是获取 store.getState().counter

```html
<button id="btn1">+1</button>
<span id="count">0</span>
<button id="btn2">-1</button>
<script src="./myRedux.js"></script>
<script src="./middlewares/logger.js"></script>
<script src="./middlewares/thunk.js"></script>
<script>
    function enhancer(createStore) {

        return function (reducer, preloadedState) {
            let store = createStore(reducer, preloadedState)
            console.log(store)

            let dispatch = store.dispatch

            function _dispatch(action) {
                if (typeof action === 'function'){
                    return action(dispatch)
                }
                dispatch(action)
            }

            return {
                ...store,
                dispatch: _dispatch
            }
        }
    }

    function counterReducer(state, action) {
        switch (action.type) {
            case 'increment':
                return state + 1
            case 'decrement':
                return state - 1
            default:
                return state
        }
    }

    let rootReducer = combineReducers({counter: counterReducer})

    const store = createStore(rootReducer, {counter: 100}, applyMiddleware(logger, thunk))
    console.log(store.getState())

    function increment() {
        return {type: 'increment'}
    }

    function decrement() {
        return {type: 'decrement'}
    }

    let actions = bindActionCreators({increment, decrement}, store.dispatch)

    document.getElementById('btn1').onclick = () => {
        // logger => thunk => reducer
        // store.dispatch({type: 'increment'})
        // store.dispatch(dispatch => {
        //     setTimeout(() => {
        //         dispatch({type: 'increment'})
        //     }, 1000)
        // })

        actions.increment()
    }
    document.getElementById('btn2').onclick = () => {
        // store.dispatch({type: 'decrement'})
        actions.decrement()
    }

    store.subscribe(() => {
        document.getElementById('count').innerText = store.getState().counter
    })
</script>
```

