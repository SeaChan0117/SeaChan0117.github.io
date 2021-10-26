---
title: 前端工程化之 Gulp
date: 2021-06-11 22:07:40
categories:
- 大前端
- 工程化
tags:
- gulp
cover: https://tse1-mm.cn.bing.net/th/id/R-C.a04c6d4902fe140cf7df7207212e1f4d?rik=DTzKlZiCsyHl1w&riu=http%3a%2f%2fwww.michaellerner.net%2fwp-content%2fuploads%2f2016%2f09%2fgulp-logo.jpg&ehk=S79Nwe6ktUUU00%2bq29NiHTeBz4HolYy5q8oZqO34Rho%3d&risl=&pid=ImgRaw&r=0
---
![gulp](bg.jpg)

在前端项目日益复杂的今天，开发效率的提升显得尤为重要，大前端大势所趋，项目工程化也成为一个合格前端开发者必不可缺的技能之一。

前端项目工程化能解决一系列开发时遇到的问题，包括：
<!--more-->
传统语言或语法带来的弊端；
无法使用模块化或组件化的编程方式去组织代码；
大量重复的机械工作；
很难的代码风格统一、质量保证；
开发过程中依赖后端服务接口支持；
整体依赖后端，需要服务运行起整个项目；
作为一个没接触过Gulp的小白，现在开始学习最流行的前端工程化工具Gulp，干巴得！

# Gulp基本使用
特点：高效易用；

使用 `Gulp` 过程简单，先在项目中安装一个 `Gulp` 的开发依赖，然后在项目根目录 `package.json` 文件中添加一个 `gulpfile.js` 的文件，用于编写一些需要 `Gulp` 自动执行的任务，完成后可以使用 `Gulp` 提供的 `cli` 去运行这些任务。

初始化项目文件 `package.json` ，执行 `yarn add gulp --dev`，将 `gulp` 作为开发依赖安装，这个过程会同时安装一个叫 `gulp-cli` 的模块，此时在 `node_modules` 下会有一个 `gulp` 的命令，用于后续去构建我们定义的任务；

![基本使用](基本使用.jpg)

在项目根目录创建一个 `gulpfile.js` 文件，接下来我们在这个文件中去定义一些用于构建的任务；该文件就是 `gulp` 入口文件，由于是运行在 `nodeJS` 环境当中，所以可以使用 `commonJS` 规范，通过导出函数成员的方式去定义构建任务；

```js
exports.foo = done => {
    console.log('foo task working');
    done()    // 最新的规范中，任务都是异步的，需要手动标识任务完成
}
```

然后执行 `yarn gulp foo` 执行任务，如果任务名是 `default` 则不用加任务名，直接 `yarn gulp` 执行；

![](01.jpg)

![](02.jpg)

在 `gulp4.0` 以前，需要引入 `gulp` 模块来创建任务：

```js
const gulp = require('gulp')
gulp.task('bar', done => {       // bar 任务名，task方法注册任务
    console.log('bar task working');
    done()
})
```

# Gulp组合任务

`Gulp` 提供了 `series` 和 `parallel` 两个 API 用于创建**串行任务**和**并行任务**；

当两个任务之间互不依赖时，就可使用并行任务提高构建效率，比如 `CSS` 和 `JS` 文件的构建是互不相关的。

```js
// 组合任务
const {series, parallel} = require('gulp')
const task1 = done => {
    setTimeout(() => {
        console.log('task1 working');
        done()
    }, 1000)
}
const task2 = done => {
    setTimeout(() => {
        console.log('task2 working');
        done()
    }, 1000)
}
const task3 = done => {
    setTimeout(() => {
        console.log('task3 working');
        done()
    }, 1000)
}

exports.seriesTask = series(task1, task2, task3)
exports.parallelTask = parallel(task1, task2, task3)
```

![组合任务](组合任务.jpg)

# gulp异步任务

当执行异步任务时，需要通知gulp任务结束，其中有以下几种最常用的方法：

## 1 通过回调的方式解决；

```js
// 回调函数
exports.callback = done => {
    console.log('callback task');
    done()
}

exports.callback_error = done => {
    console.log('callback error task');
    done(new Error('task error!'))      // 执行错误，多个任务同时执行时，后续将不再执行
}
```

## 2 promise 方法；

```js
// promise方法
exports.promise = () => {
    console.log('promise task');
    return Promise.resolve()    // resolve不用传参，gulp忽略
}

exports.promise_error = () => {
    console.log('promise task');
    return Promise.reject()    // 不用传参，gulp忽略
}
```

## 3 Async 和 Await 方法；

```js
// Async/Await  =============== promise语法糖
const timeout = time => {
    return new Promise(resolve => {
        setTimeout(resolve, time)
    })
}
exports.async = async () => {
    await timeout(1000)
    console.log('async task');
}
```

## 4 stream 方式，最常见，因为构建过程大多是在处理文件；

```js
// stream方式
const fs = require('fs')
exports.stream = () => {
    const readStream = fs.createReadStream('package.json')
    const writeStream = fs.createWriteStream('temp.txt')
    readStream.pipe(writeStream)
    return readStream
}
// 下边写法结果相同
exports.stream = done => {
    const readStream = fs.createReadStream('package.json')
    const writeStream = fs.createWriteStream('temp.txt')
    readStream.pipe(writeStream)
    readStream.on('end', () => {
        done()
    })
}
```

执行 `yarn gulp stream` 后，在根目录下生成拷贝了 `package.json` 内容的 `temp.txt` 文件。

# Gulp构建过程核心工作原理

![手动打包过程](手动打包过程.jpg)

![gulp模拟手动打包过程（流操作）](gulp模拟手动打包过程（流操作）.jpg)

实际操作演示，一个简单的 `css` 文件打包过程：

```js
const fs = require('fs')
const {Transform} = require('stream')
exports.default = () => {
    // 文件读取流
    const read = fs.createReadStream('normalize.css')
    // 文件写入流
    const write = fs.createWriteStream('normalize.min.css')
    // 文件转换流对象
    const transform = new Transform({
        transform(chunk, encoding, callback) {
            // 核心转换过程
            // chunk ==> 读取流中读取的内容（Buffer），toString转为字符串
            const input = chunk.toString()
            // 清除字符串中的空格和注释
            const output = input.replace(/\s+/g, '').replace(/\/\*.+?\*\//g, '')
            callback(null, output) // 键转换后的字符串返回出去，错误优先，没有错误第一个参数为null
        }
    })
    // 把读取出来的文件流导入到写入流当中
    read
        .pipe(transform)    // 转换
        .pipe(write)        // 写入
    return read
}
```

# Gulp 文件操作 API + 插件的使用

`gulp` 提供了专门读取和写入文件流的 `API` ，针对文件加工的转换流绝大多数可以使用独立的插件，实际通过 `gulp` 去创建构建任务的流程是通过 `src` 方法创建一个读取流，然后借助插件提供的转换流来实现文件加工，最后通过 `gulp` 提供的 `dest` 方法创建写入流写入到目标文件；

安装 `gulp-clean-css` 和 `gulp-rename` 插件来实现一个 `css` 文件打包过程：

```js
const {src, dest} = require('gulp')
const cleanCSS = require('gulp-clean-css')
const rename = require('gulp-rename')
exports.default = () => {
    return src('src/*.css') // 读取
        .pipe(cleanCSS())   // 转换压缩
        .pipe(rename({extname: '.min.css'}))        // 重命名扩展名
        .pipe(dest('dist'))         // 写入
}
```

![](03.jpg)

# 总结：

第一次系统性的开始学习构建工具，深感强大，自己用过 `webpack` 仅是用于开发方法，按照流程起个项目只顾业务，最后打个包扔给后台，完全没有学习的过程。本次 `gulp` 学习开始，使用的东西要知其然知其所以然，当然 `gulp` 只是初探，还有很多要学的，下一篇咱开始一个实际自动化构建过程去了解更多关于 `gulp` 的东西。console.log('gulp，加油！')

