---
title: 搭建自己的 Vue-SSR
date: 2021-06-19 17:01:18
categories:
- 大前端
- SSR
tags:
- SSR
- 服务端渲染
---
# SSR（Server Side Render）

## SSR

即服务端渲染，现在一些基于模板的框架编写应用，展示在浏览器界面之前要先渲染为浏览器所知的 HTML 文档，顾名思义，SSR 就是把这个渲染过程搬到了服务端，服务端渲染为 HTML 后再返回给浏览器，浏览器就能直接展示。简单来看，模板在浏览器解析或者一些初始化数据的展示，都需要涉及解析处理等，这一定离不开 JS，浏览器初始化就需要先加载 JS，但是在服务端渲染，这一步基本就少，所以这是可以减少初始化渲染时间的，即所谓的首屏渲染。简单来看，这确实是能减少首屏渲染时间，但实际首屏渲染可不止这么简单。

<!--more-->

## [Vue 官方对 SSR 的解释](https://ssr.vuejs.org/zh)

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

## [为什么使用服务器端渲染](https://ssr.vuejs.org/zh)

与传统 SPA (单页应用程序 (Single-Page Application)) 相比，服务器端渲染 (SSR) 的优势主要在于：

- 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。

  请注意，截至目前，Google 和 Bing 可以很好对同步 JavaScript 应用程序进行索引。在这里，同步是关键。如果你的应用程序初始展示 loading 菊花图，然后通过 Ajax 获取内容，抓取工具并不会等待异步完成后再行抓取页面内容。也就是说，如果 SEO 对你的站点至关重要，而你的页面又是异步获取内容，则你可能需要服务器端渲染(SSR)解决此问题。

- 更快的内容到达时间 (time-to-content)，特别是对于缓慢的网络情况或运行缓慢的设备。无需等待所有的 JavaScript 都完成下载并执行，才显示服务器渲染的标记，所以你的用户将会更快速地看到完整渲染的页面。通常可以产生更好的用户体验，并且对于那些「内容到达时间(time-to-content) 与转化率直接相关」的应用程序而言，服务器端渲染 (SSR) 至关重要。

使用服务器端渲染 (SSR) 时还需要有一些权衡之处：

- 开发条件所限。浏览器特定的代码，只能在某些生命周期钩子函数 (lifecycle hook) 中使用；一些外部扩展库 (external library) 可能需要特殊处理，才能在服务器渲染应用程序中运行。
- 涉及构建设置和部署的更多要求。与可以部署在任何静态文件服务器上的完全静态单页面应用程序 (SPA) 不同，服务器渲染应用程序，需要处于 Node.js server 运行环境。
- 更多的服务器端负载。在 Node.js 中渲染完整的应用程序，显然会比仅仅提供静态文件的 server 更加大量占用 CPU 资源 (CPU-intensive - CPU 密集)，因此如果你预料在高流量环境 (high traffic) 下使用，请准备相应的服务器负载，并明智地采用缓存策略。

下面，根据 Vue 官方的 SSR 方案，以及学习过程中的记录，整理了怎样实现一套完整的 Vue-SSR 。

# 怎样服务端渲染一个 Vue 实例
初始化自己的项目，然后安装 **vue** 和 **vue-server-renderer**;
```
$ yarn add vue vue-server-renderer
```
代码根路径下编写 server.js，用于服务端执行，导入 vue 和 vue-server-renderer，Vue 实例模板 app，绑定了数据 message，vue-server-renderer 调用 **createRenderer** 方法，创建 renderer 实例，renderer .renderToString 方法能将 app 渲染为 html，控制台用 node 执行 server.js 能看到打印出渲染过的 HTML 标签内容，```<div id="app" data-server-rendered="true"><h1>你好啊 Vue SSR</h1></div>```，即 message 已经被渲染到了 HTML中。
```javascript
const Vue = require('vue')
const renderer = require('vue-server-renderer').createRenderer()

const app = new Vue({
    template: `
        <div id="app">
            <h1>{{ message }}</h1>
        </div>
    `,
    data: {
        message: '你好啊 Vue SSR'
    }
})

renderer.renderToString(app, (err, html) => {
    if (err) throw err

    console.log(html)
})
```

# 将模板发送给浏览器

上面我们已经在服务端渲染好了一个简单的 Vue 实例，接下来要将渲染好的 HTML 发送给浏览器，让浏览器直接呈现。和 web 交互，在 node 中我们使用 express，首先安装 ```yarn add express```。编写代码如下，此处启动一个服务在本地 3000 端口，当访问路径 '/' 时，返回渲染好的 HTML。

通过命令 ```nodemon server.js``` 启动，浏览器打开 [http://localhost:3000/](http://localhost:3000/)，看到页面展示了渲染好的内容。

```javascript
const Vue = require('vue')
const renderer = require('vue-server-renderer').createRenderer()
const express = require('express')
const server = express()

server.get('/', (req, res) => {
    const app = new Vue({
        template: `
        <div id="app">
            <h1>{{ message }}</h1>
        </div>
    `,
        data: {
            message: '你好啊 Vue SSR'
        }
    })

    renderer.renderToString(app, (err, html) => {
        if (err) {
            res.status(500).end('Internal Server Error.')
        }
        // 设置编码，防止中文等乱码
        res.setHeader('Content-Type', 'text/html; charset=utf8')
        res.end(html)
    })
})

server.listen(3000, () => {
    console.log('server is running at port 3000')
})
```

![渲染结果展示到浏览器](2021-06-20_221437.png)

# HTML 模板

上述代码中，在返回 HTML 给浏览器前，对响应头设置了相应的编码，也可以返回一个完整的 HTML 页面字符串，将渲染好的字符串插入其中，这里把完整的 HTML 提出来作为一个单独的文件，后续还可对其进行其他操作。

## 使用 HTML 模板

在项目根目录创建文件 index.template.html 。注意，里边在 body 中添加一个标记注释（vue-ssr-outlet），后续会被渲染内容找到替换在此。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Vue SSR</title>
</head>
<body>
<!--vue-ssr-outlet-->
</body>
</html>
```

然后在创建 renderer 的方法中传入 template，具体代码如下：

```javascript
const Vue = require('vue')
const express = require('express')
const fs = require('fs')

const renderer = require('vue-server-renderer').createRenderer({
    template: fs.readFileSync('./index.template.html', 'utf-8')
})

const server = express()

server.get('/', (req, res) => {
    const app = new Vue({
        template: `
            <div id="app">
                <h1>{{ message }}</h1>
            </div>
        `,
        data: {
            message: '你好啊 Vue SSR'
        }
    })

    renderer.renderToString(app, (err, html) => {
        if (err) {
            res.status(500).end('Internal Server Error.')
        }
        res.end(html)
    })
})

server.listen(3000, () => {
    console.log('server is running at port 3000')
})
```

此时已经删除响应头设置的编码，刷新浏览器后，页面依然正常加载。

## 模板中使用外部数据

HTML 模板中还可以使用其他的外部数据，在 **renderer.renderToString** 方法中传入第二个可选的参数，比如我们来更改一下页面的 Title 和添加一个 meta 标签，在 HTML 模板中使用差值表达式，**插入整个标签需要三层花括号**，重启服务。

```javascript
renderer.renderToString(app, {
    title: "外部数据title",
    meta: `<meta name="description" content="我是 vue ssr">`
},(err, html) => {
    if (err) {
        res.status(500).end('Internal Server Error.')
    }
    res.end(html)
})
```

```html
<head>
    <meta charset="UTF-8">
    {{{ meta }}}
    <title>{{ title }}</title>
</head>
```

![模板中使用外部数据](2021-06-20_225808.png)

# 构建配置

## 基本思路

# 相关链接：

[https://ssr.vuejs.org/](https://ssr.vuejs.org/);  

