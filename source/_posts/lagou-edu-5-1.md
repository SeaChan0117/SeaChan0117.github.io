---
title: 拉钩学习 5-1 作业
date: 2021-10-04 14:28:15
categories:
- 大前端
- node
tags:
- 大前端
- node
---

拉钩学习 5-1 作业
<!--more-->

# 简单题
## 简述 Node.js 的特点以及适用的场景
### Node.js 特点：
1. I/O 异步：
> 在 Node.js 中，文件读写和网络请求等，大多操作都是异步方式进行调用，异步 I/O 即意味着在 I/O 操作时无需等待之前的操作结束，在编程模型上可以提升效率；
2. 事件与回调：
> 在 Node.js 中，很多功能是继承自 events 模块实现的，所以事件得到了广泛的应用，而事件的处理基本都是依赖回调函数完成的，正因如此，Node 中的回调也就无处不在；
> 由于异步代码的广泛应用，其中又有一套完整的事件循环机制负责处理事件的循环和调度；
3. 单线程：
> Node.js 保持了JavaScript在浏览器中单线程的特点，而且在 Node 中，JavaScript 与其余线程是无法共享任何状态的；
> 所以 Node 无法体现多核 CPU 的优势，处理 CUP 密集型任务时会占用过多的CPU

### Node.js 适用场景：
基于以上 Node.js 的特点得出，非阻塞的异步 I/O 操作让 Node.js 适用于 I/O 密集型的高并发请求应用，
比如大型应用中前后端之间再搭建一个 BFF 层，实时聊天程序等；


## 简述 Buffer 的使用.包括多种创建方式。实例方法，静态方法
Buffer 是 Node.js 中的一个全局变量之一，意思是缓冲区，即一片内存空间（V8内存之外，Node 使用，V8 做 GC 回收），用于操作二进制数据；
1. Buffer 创建方式：
    * alloc：创建指定大小字节的 buffer
    * allocUnsafe：创建指定大小字节的 buffer（不安全）
    * from：接收数据，创建 buffer
    
2. Buffer 实例方法：
    * fill：使用数据填充 buffer
    * write：向 buffer 中写入数据
    * toString：从 buffer 中提取数据，并按指定格式拿到数据
    * slice：向数组一样截取 buffer
    * indexOf：在 buffer 中查找数据
    * copy：拷贝 buffer 中的数据
    
3. Buffer 静态方法
    * concat：将多个 buffer 拼接成一个新的 buffer
    * isBuffer：判断当前数据是否为 buffer

## 写出5个以上文件操作的API，并且用文字说明其功能
1. 异步打开文件：fs.open(path[, flags[, mode]], callback)
```javascript
import fs = 'fs'
fs.open('test.txt', 'r+', (err, fd) => {
    if (err) throw new Error('文件打开失败')
    console.log('文件打开成功')
})
```
2. 异步获取文件信息：fs.stat(path, callback)
```javascript
import fs = 'fs'
fs.stat('test.txt', 'r+', (err, stats) => {
    if (stats.isFile()) {
        console.log('文件')
    } else {
        console.log('文件夹')
    }
})
```
3. 异步地读取文件的全部内容：fs.readFile(path[, options], callback)
```javascript
import fs = 'fs'
fs.readFile('test.txt', (err, data) => {
    if (err) throw err
    console.log(data)
})
```
4. 异步写入文件：fs.writeFile(file, data[, options], callback)
```javascript
import fs = 'fs'
fs.writeFile('test.txt', '我是写入的内容字符串',  err => {
    if (err) throw err
    console.log('写入成功')
})
```
5. 异步地创建目录：fs.mkdir(path[, options], callback)
```javascript
import fs = 'fs'
fs.mkdir('/a/b/c', { recursive: true }, err => {
    if (err) throw err
    console.log('创建成功')
})
```

## 简述使用流操作的优势，以及Node中流的分类
流操作的优势：
* 时间效率：流的分段处理可以同时操作多个数据 chunk
* 空间效率：同一时间流无须占据大内存空间
* 使用方便：流配合管道，扩展程序变得简单

Node.js 中流的分类：
* Readable：可读流，能实现数据的读取
* Writeable：可写流，能实现数据的写操作
* Duplex：双工流，既可读又可写
* Transform：转换流，可读可写，还能实现数据转换


## 在数据封装与解封装过程中，针对应用层、传输层、网络层、数据链路层、物理层5层分别做了什么事情
数据封装过程：
* 应用层产出传输的数据；
* 数据传向传输层，在这一层最常见的就是 TCP 和 UDP 协议，这两个协议都是基于端口的，端口作用就是在主机上确定应用进程，所以数据在这一层就会被包裹上目标应用端口和应用在当前的源端口；
* 数据传向网络层，主机处于不同的网络里的，所以需要通过IP协议来确定目标主机在网络，因此数据在这一层会被包裹上目标 IP,端口，源主机 IP，端口，确定某一个网络；
* 数据链路层，通过MAC地址来完成寻址操作，数据会被包裹上目标 Mac 和源 Mac ,到此一条完整的数据封装完成，递给网络；
* 物理层，网线不能识别二进制数据，需要通过网卡调制之后，会形成高低电压，到达目标主机的网卡；

数据的解封过程和封装过程经过的层级顺序刚好反过来：
* 目标的物理层 首先做数据的解调，将电压变为二进制，向上层传递至链路层；
* 链路层会进行分析目标的 Mac 地址是不是当前主机的 Mac 地址，是的话继续向上传递网络层；
* 网络层解析目标 IP 是不是自己的 IP，如果是继续拆包，向上传输至传输层；
* 传输层确定当前的目标端口是不是自己的，是再拆解数据，并传输至应用层；
* 到达应用层，这时候主机接收到另外一台主题的数据；

# 代码题
## 统计指定目录中文件总大小。要考虑目录中还有子目录的情况。可以同步编码,异步更好
```javascript
/**
 * 统计文件或文件夹所占字节数
 * node 执行该文件，在命令询问文件或文件夹路径，输入后回车
 */
const fs = require('fs')
const path = require('path')
const readline = require('readline')
const {promisify} = require('util')

const stat = promisify(fs.stat)
const readdir = promisify(fs.readdir)

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
})

rl.question(`Please input file or dictionary path：`, async pathStr => {
    await getSize(pathStr, (err, size) => {
        if (err) {
            console.log(err)
            rl.close()
            return
        }
        console.log(`Files size is ${size} bytes`)
        rl.close()
    })
})

async function getSize(dirPath, done) {
    let size = 0
    let err

    async function calcSize(dirPath) {
        try {
            const statObj = await stat(dirPath)
            if (statObj.isFile()) {
                size += statObj.size
            } else {
                let dirs = await readdir(dirPath)
                dirs = dirs.map(item => path.join(dirPath, item))
                for (let i = 0; i < dirs.length; i++) {
                    await calcSize(dirs[i])
                }
            }
        } catch (e) {
            err = e
        }
    }

    await calcSize(dirPath)
    done(err, size)
}
```

## 编写单向链表类并且实现队列的入列出列操作
```javascript
/**
 * 链表节点
 */
class Node {
    constructor(element, next) {
        this.element = element
        this.next = next
    }
}

/**
 * 单向链表
 */
class LinkedList {
    constructor(head, size) {
        this.head = null
        this.size = 0
    }
    // 添加节点
    add(index, element) {
        if (arguments.length === 1) {
            element = index
            index = this.size
        }
        if (index < 0 || index > this.size) throw new Error('over size')
        if (index === 0) {
            let head = this.head
            this.head = new Node(element, head)
        } else {
            let prevNode = this._getNode(index - 1)
            prevNode.next = new Node(element, prevNode.next)
        }
        this.size++
    }
    // 移除节点
    remove(index) {
        let rmNode = null
        if (index === 0) {
            rmNode = this.head
            if (!rmNode) {
                return undefined
            }
            this.head = rmNode.next
        } else {
            const preNode = this._getNode(index - 1)
            rmNode = preNode.next
            preNode.next = rmNode.next
        }
        this.size--
        return rmNode
    }
    // 修改节点
    setNode(index, element) {
        const node = this._getNode(index)
        node.element = element
    }
    // 获取节点
    getNode(index) {
        return this._getNode(index)
    }
    clear() {
        this.head = null
        this.size = 0
    }

    _getNode(index) {
        if (index < 0 || index >= this.size) throw new Error('over size')
        let node = this.head
        for (let i = 0; i < index; i++) {
            node = node.next
        }
        return node
    }
}

/**
 * 队列
 */
class Queue {
    constructor() {
        this.likedList = new LinkedList()
    }
    // 进队列
    enQueue(data) {
        this.likedList.add(data)
    }
    // 出队列
    deQueue() {
        return this.likedList.remove(0)
    }
}

// 队列实例
const q = new Queue()
q.enQueue('node1')
q.enQueue('node2')

const data = q.deQueue()
console.log(data) // node1 对应的节点，先进先出
```

## 基于Node写出一静态服务器。接收请求并且响应特定目录(服务器目录)中的html、css、js、图片等资源
https://github.com/SeaChan0117/node-static-server


