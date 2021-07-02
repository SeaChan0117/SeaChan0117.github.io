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

## 什么是 SSR

SSR 是 Server Side Render 简称，就是在服务端进行渲染生成 HTML 文件，浏览器显示生成的 HTML 文件， 补充：我们传统使用的属于 CSR 是 Client Side Render ，页面上的内容是我们加载的 JS 文件渲染出来的，文件运行在浏览器上面。

<!--more-->

## [Vue 官方对 SSR 的解释](https://ssr.vuejs.org/zh)

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在**服务器**和**客户端**上运行。

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

上边得到的只是一个静态的 HTML 模板页面，如果在渲染前加上客户端交互的功能（如 v-model 双向绑定、点击事件等），你会发现并不能生效，其实查看页面响应也能知道，它并没有包含客户端交互所需的 JS。官方为我们提供了一个基本的构建思路去处理这个问题

## 基本思路

Vue SSR 官方这样给出解释，对于客户端应用程序和服务器应用程序，我们都要使用 webpack 打包 - 服务器需要「服务器 bundle」然后用于服务器端渲染(SSR)，而「客户端 bundle」会发送给浏览器，用于混合静态标记。

![构建思路](pack-steps.png)

上边的图我们看出，针对构建结构，目前我们只有服务端入口和出口，即只有服务端渲染，所以只能拿到静态的 HTML 字符串，还需要一个用于客户端的入口和出口，生成一个客户端的 Bundle，用于接管服务端渲染的页面，并激活为一个动态页面。

## 代码结构

针对上述图解，官方给出一个基本的代码结构，我们参考来实现。

```sh
src
├── components
│   ├── Foo.vue
│   ├── Bar.vue
│   └── Baz.vue
├── App.vue
├── app.js # 通用 entry(universal entry)
├── entry-client.js # 仅运行于浏览器
└── entry-server.js # 仅运行于服务器
```

新建 src 目录；  

### App.vue

```vue
<template>
    <div id="app">
        <h1>{{ message }}</h1>
        <h2> 客户端动态交互 </h2>
        <div>
            <input type="text" v-mode="message">
        </div>
        <div>
            <button @click="onClick">Click Test</button>
        </div>
    </div>
</template>
<script>
    export default {
        name: "App",
        data() {
            return {
                message: '我是 Vue SSR'
            }
        },
        methods: {
            onClick() {
                console.log('Vue SSR')
            }
        }
    }
</script>
```

### app.js

```javascript
/**
 * 通用的启动入口
 *
 * app.js 是我们应用程序的「通用 entry」。在纯客户端应用程序中，我们将在此文件中创建根 Vue 实例，
 * 并直接挂载到 DOM。但是，对于服务器端渲染(SSR)，责任转移到纯客户端 entry 文件。app.js 简单地使用 export 导出一个 createApp 函数：
 */
import Vue from 'vue'
import App from './App.vue'

// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp () {
    const app = new Vue({
        // 根实例简单的渲染应用程序组件。
        render: h => h(App)
    })
    return { app }
}
```

### entry-client.js

```javascript
/**
 * 客户端 entry 只需创建应用程序，并且将其挂载到 DOM 中
 */
import { createApp } from './app'
// 客户端特定引导逻辑……
const { app } = createApp()
// 这里假定 App.vue 模板中根元素具有 `id="app"`
app.$mount('#app')
```

### entry-server.js

```javascript
/**
 * 服务器 entry 使用 default export 导出函数，并在每次渲染中重复调用此函数。
 * 此时，除了创建和返回应用程序实例之外，它不会做太多事情 - 但是稍后我们将在此执行服务器端路由匹配 (server-side route matching) 和数据预取逻辑 (data pre-fetching logic)。
 */
import { createApp } from './app'

export default context => {
    const { app } = createApp()
    // TODO：服务端路由处理，数据预取...
    return app
}
```

以上完成了最基础的代码结构，但目前还不能运行测试， 这只是源代码结构，还需要通过 webpack 打包。

# webpack 打包构建

## 安装依赖

### 生产依赖

```sh
yarn add vue vue-server-renderer express cross-env
```

| 包                  | 说明                                |
| ------------------- | ----------------------------------- |
| vue                 | Vue.js 核心库                       |
| vue-server-renderer | Vue 服务端渲染工具                  |
| express             | 基于 Node 的 Web 服务框架           |
| cross-env           | 通过 npm scripts 设置跨平台环境变量 |

###  开发依赖

```sh
yarn add -D webpack webpack-cli webpack-merge webpack-node-externals @babel/core @babel/plugin-transform-runtime @babel/preset-env babel-loader css-loader url-loader file-loader rimraf vue-loader vue-template-compiler friendly-errors-webpack-plugin
```

| 包                                                           | 说明                                       |
| ------------------------------------------------------------ | :----------------------------------------- |
| webpack                                                      | webpack 核心包                             |
| webpack-cli                                                  | webpack 的命令行工具                       |
| webpack-merge                                                | webpack 配置信息合并工具                   |
| webpack-node-externals                                       | 排除 webpack 中的 Node 模块                |
| rimraf                                                       | 基于 Node 封装的一个跨平台 **rm -rf** 工具 |
| friendly-errors-webpack-plugin                               | 友好的 webpack 错误提示                    |
| @babel/core<br/>@babel/plugin-transform-runtime<br/>@babel/preset-env<br/>babel-loader | Babel 相关工具                             |
| vue-loader<br/>vue-template-compiler                         | 处理 .vue 资源                             |
| file-loader                                                  | 处理字体资源                               |
| css-loader                                                   | 处理 CSS 资源                              |
| url-loader                                                   | 处理图片资源                               |

## webpack  配置文件

### 初始化 webpack 打包配置文件

服务器端渲染 (SSR) 项目的配置大体上与纯客户端项目类似，建议将配置分为三个文件：*base*, *client* 和 *server*。基本配置 (base config) 包含在两个环境共享的配置，例如，输出路径 (output path)，别名 (alias) 和 loader。服务器配置 (server config) 和客户端配置 (client config)，可以通过使用 [webpack-merge](https://github.com/survivejs/webpack-merge) 来简单地扩展基本配置。

```sh
build
├── webpack.base.config.js # 公共配置
├── webpack.client.config.js # 客户端打包配置文件
└── webpack.server.config.js # 服务端打包配置文件
```

### webpack.base.config.js

```javascript
/**
 * 公共配置
 */
const VueLoaderPlugin = require('vue-loader/lib/plugin')
const path = require('path')
const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin')
const resolve = file => path.resolve(__dirname, file)
const isProd = process.env.NODE_ENV === 'production'
module.exports = {
    mode: isProd ? 'production' : 'development',
    output: {
        path: resolve('../dist/'),
        publicPath: '/dist/',
        filename: '[name].[chunkhash].js'
    },
    resolve: {
        alias: {
            // 路径别名，@ 指向 src
            '@': resolve('../src/')
        },
        // 可以省略的扩展名
        // 当省略扩展名的时候，按照从前往后的顺序依次解析
        extensions: ['.js', '.vue', '.json']
    },
    devtool: isProd ? 'source-map' : 'eval-cheap-module-source-map',
    module: {
        rules: [
            // 处理图片资源
            {
                test: /\.(png|jpg|gif)$/i,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 8192,
                        },
                    },
                ],
            },
            // 处理字体资源
            {
                test: /\.(woff|woff2|eot|ttf|otf)$/,
                use: [
                    'file-loader',
                ],
            },
            // 处理 .vue 资源
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            // 处理 CSS 资源
            // 它会应用到普通的 `.css` 文件
            // 以及 `.vue` 文件中的 `<style>` 块
            {
                test: /\.css$/,
                use: [
                    'vue-style-loader',
                    'css-loader'
                ]
            },
        // CSS 预处理器，参考：https://vue-loader.vuejs.org/zh/guide/preprocessors.html
        // 例如处理 Less 资源
        // {
        // test: /\.less$/,
        // use: [
        // 'vue-style-loader',
        // 'css-loader',
        // 'less-loader'
        // ]
        // },
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new FriendlyErrorsWebpackPlugin()
    ]
}

```

### webpack.client.config.js

```javascript
/**
 * 客户端打包配置
 */
const {merge} = require('webpack-merge')
const baseConfig = require('./webpack.base.config.js')
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')
module.exports = merge(baseConfig, {
    entry: {
        app: './src/entry-client.js'
    },
    module: {
        rules: [
            // ES6 转 ES5
            {
                test: /\.m?js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env'],
                        cacheDirectory: true,
                        plugins: ['@babel/plugin-transform-runtime']
                    }
                }
            },
        ]
    },
    // 重要信息：这将 webpack 运行时分离到一个引导 chunk 中，
    // 以便可以在之后正确注入异步 chunk。
    optimization: {
        splitChunks: {
            name: "manifest",
            minChunks: Infinity
        }
    },
    plugins: [
        // 此插件在输出目录中生成 `vue-ssr-client-manifest.json`。
        new VueSSRClientPlugin()
    ]
})
```

### webpack.server.config.js

```javascript
/**
 * 服务端打包配置
 */
const {merge} = require('webpack-merge')
const nodeExternals = require('webpack-node-externals')
const baseConfig = require('./webpack.base.config.js')
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin')
module.exports = merge(baseConfig, {
    // 将 entry 指向应用程序的 server entry 文件
    entry: './src/entry-server.js',
    // 这允许 webpack 以 Node 适用方式处理模块加载
    // 并且还会在编译 Vue 组件时，
    // 告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
    target: 'node',
    output: {
        filename: 'server-bundle.js',
    // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
        libraryTarget: 'commonjs2'
    },
    // 不打包 node_modules 第三方包，而是保留 require 方式直接加载
    externals: [nodeExternals({
    // 白名单中的资源依然正常打包
        allowlist: [/\.css$/]
    })],
    plugins: [
    // 这是将服务器的整个输出构建为单个 JSON 文件的插件。
    // 默认文件名为 `vue-ssr-server-bundle.json`
        new VueSSRServerPlugin()
    ]
})
```



### 配置 **npm scripts**

```json
"scripts": {
"build:client": "cross-env NODE_ENV=production webpack --config build/webpack.client.config.js",
"build:server": "cross-env NODE_ENV=production webpack --config build/webpack.server.config.js",
"build": "rimraf dist && npm run build:client && npm run build:server"
}
```

执行上述命令测试构建是否能正确进行。

## 启动应用

参考 [https://ssr.vuejs.org/zh/guide/bundle-renderer.html](https://ssr.vuejs.org/zh/guide/bundle-renderer.html) 我们需要将 server.js 中的 *createRenderer* 方法替换为 *createBundleRenderer* ，它接收第一个参数 serverBundle，就是上述打包生成的 vue-ssr-server-bundle.json，第二个参数中加一个参数 manifest，即打包生成的客户端文件 vue-ssr-client-manifest.json。此时这个 renderer 会自动去配置 entry-server.js 中创建的 app，所以此处删除模板配置。

```javascript
/**
* server.js
*/
const Vue = require('vue')
const express = require('express')
const fs = require('fs')

const serverBundle = require('./dist/vue-ssr-server-bundle.json')
const template = fs.readFileSync('./index.template.html', 'utf-8')
const clientManifest  = require('./dist/vue-ssr-client-manifest.json')

const renderer = require('vue-server-renderer').createBundleRenderer(serverBundle, {
    runInNewContext: false,
    template,
    clientManifest
})

const server = express()
server.use('/dist', express.static('./dist')) // 开发 dist 目录下的资源，否则找不到

server.get('*', (req, res) => {
    renderer.renderToString({
        title: "vue-ssr",
        meta: `<meta name="description" content="我是 vue ssr">`
    }, (err, html) => {
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

此时就可以启动服务( ```nodemon server.js``` )查看，文件加载 OK，测试数据双向绑定 OK。

![启动应用-数据双向绑定测试](2021-06-30_140618.png)

以上我们已经实现了 Vue-SSR 的基本功能，但是还有**数据预取**和**状态管理**等功能还未实现，在实现之前需要做一些处理帮助我们提高开发效率，即创建一个开发模式，当文件改动后，能即时编译后自动刷新到页面，保证我们开发过程中不用重新服务和手动刷新浏览器等。

# 搭建开发模式

思路：监视打包构建 --> 重新生成 Renderer 渲染器

## package.json

新增启动命令，区分生产模式启动（start）和开发模式启动（dev），生产模式传递了环境变量 NODE_ENV=production；

```json
"scripts": {
    "build:client": "cross-env NODE_ENV=production webpack --config build/webpack.client.config.js",
    "build:server": "cross-env NODE_ENV=production webpack --config build/webpack.server.config.js",
    "build": "rimraf dist && npm run build:client && npm run build:server",
    "start": "cross-env NODE_ENV=production node server.js",
    "dev": "node server.js"
  },
```

## server.js

server.js 改造将思路初始化。

```javascript
const express = require('express')
const fs = require('fs')

const isProd = process.env.NODE_ENV === 'production'
let renderer

if (isProd) {
    const serverBundle = require('./dist/vue-ssr-server-bundle.json')
    const template = fs.readFileSync('./index.template.html', 'utf-8')
    const clientManifest  = require('./dist/vue-ssr-client-manifest.json')
    renderer = require('vue-server-renderer').createBundleRenderer(serverBundle, {
        runInNewContext: false,
        template,
        clientManifest
    })
} else {
    // 开发模式 --> 监视打包构建 --> 重新生成 Renderer 渲染器
}

const server = express()
server.use('/dist', express.static('./dist')) // 开发 dist 目录下的资源，否则找不到

const render = (req, res) => {
    renderer.renderToString({
        title: "vue-ssr",
        meta: `<meta name="description" content="我是 vue ssr">`
    }, (err, html) => {
        if (err) {
            res.status(500).end('Internal Server Error.')
        }
        res.end(html)
    })
}

server.get('*', isProd
    ? render
    : (req, res) => {
        // TODO：等待有了 Renderer 渲染器后，调用 render 进行渲染
        render()
    }

)

server.listen(3000, () => {
    console.log('server is running at port 3000')
})

```

## 提取处理模块

开发模式的思路主要实现，监听源码变化；

build 目录下新增 setup-dev-server.js 用于处理监听构建，更新 Renderer；注意此处使用了 chokidar 作为监听文件变化的第三方包，记得安装相关依赖 ```yarn add chokidar```

```javascript
/**
* setup-dev-server.js
*/
const fs = require('fs')
const path = require('path')
const chokidar = require('chokidar')

module.exports = (server, callback) => {
    let ready
    const onReady = new Promise(r => ready = r)

    // 监视构建 ==> 更新 Renderer，即调用 callback
    let template, serverBundle, clientManifest

    const update = () => {
        if (template && serverBundle && clientManifest) {
            ready()
            callback(serverBundle, template, clientManifest)
        }
    }

    update()
    // 监视构建 template --> 调用 update --> 更新 Renderer 渲染器
    const templatePath = path.resolve(__dirname, '../index.template.html')
    template = fs.readFileSync(templatePath, 'utf-8')
    // 监听文件变化的方法，此处使用第三方的 chokidar fs.watch  fa.watchFile chokidar
    chokidar.watch(templatePath).on('change', () => {
        update()
    })
    
    // 监视构建 serverBundle --> 调用 update --> 更新 Renderer 渲染器
    // 监视构建 clientManifest --> 调用 update --> 更新 Renderer 渲染器

    return onReady
}
```

导入到 server.js 并使用

```javascript
const setUpDevServer = require('./build/setup-dev-server')
...
} else {
    // 开发模式 --> 监视打包构建 --> 重新生成 Renderer 渲染器
    onReady = setUpDevServer(server, (serverBundle, template, clientManifest) => {
        renderer = createBundleRenderer(serverBundle, {
            template,
            clientManifest
        })
    })
}
...
```

## 服务端监视打包

webpack 编译器监视打包文件发生变化后，执行 update 方法；

```javascript
/**
* setup-dev-server.js
*/
const fs = require('fs')
const path = require('path')
const chokidar = require('chokidar')
const webpack = require('webpack')

const resolve = filePath => path.resolve(__dirname, filePath)

module.exports = (server, callback) => {
    let ready
    const onReady = new Promise(r => ready = r)

    // 监视构建 ==> 更新 Renderer，即调用 callback
    let template, serverBundle, clientManifest

    const update = () => {
        if (template && serverBundle && clientManifest) {
            ready()
            callback(serverBundle, template, clientManifest)
        }
    }

    update()
    // 监视构建 template --> 调用 update --> 更新 Renderer 渲染器
    const templatePath = resolve('../index.template.html')
    template = fs.readFileSync(templatePath, 'utf-8')
    // 监听文件变化的方法，此处使用第三方的 chokidar fs.watch  fa.watchFile chokidar
    chokidar.watch(templatePath).on('change', () => {
        update()
    })

    // 监视构建 serverBundle --> 调用 update --> 更新 Renderer 渲染器
    const serverConfig = require('./webpack.server.config')
    const serverCompiler = webpack(serverConfig)
    serverCompiler.watch({}, (err, stats) => {
        if (err) throw err
        if (stats.hasErrors()) return
        serverBundle = JSON.parse(
            fs.readFileSync(resolve('../dist/vue-ssr-server-bundle.json'), 'utf-8')
        )
        update()
    })

    // 监视构建 clientManifest --> 调用 update --> 更新 Renderer 渲染器

    return onReady
}
```

## 把数据写入内存

以上处理中我们会发现涉及到文件读写，在开发模式下，频繁改动文件，会造成频繁读写磁盘，效率是很低的，此处把构建的数据放到内存中用于读写，提高构建速度。参考 [webpack 方案](https://webpack.js.org/api/node/#custom-file-systems) ，可以使用 三方的 memfs 来处理，也可以使用 webpack 提供的 webpack-dev-middleware ，此处使用后者，```yarn add  webpack-dev-middleware -D``` 。

```javascript
/**
* setup-dev-server.js
*/
const fs = require('fs')
const path = require('path')
const chokidar = require('chokidar')
const webpack = require('webpack')
const devMiddleware = require('webpack-dev-middleware')

const resolve = filePath => path.resolve(__dirname, filePath)

module.exports = (server, callback) => {
    let ready
    const onReady = new Promise(r => ready = r)

    // 监视构建 ==> 更新 Renderer，即调用 callback
    let template, serverBundle, clientManifest

    const update = () => {
        if (template && serverBundle && clientManifest) {
            ready()
            callback(serverBundle, template, clientManifest)
        }
    }

    update()
    // 监视构建 template --> 调用 update --> 更新 Renderer 渲染器
    const templatePath = resolve('../index.template.html')
    template = fs.readFileSync(templatePath, 'utf-8')
    // 监听文件变化的方法，此处使用第三方的 chokidar fs.watch  fa.watchFile chokidar
    chokidar.watch(templatePath).on('change', () => {
        update()
    })

    // 监视构建 serverBundle --> 调用 update --> 更新 Renderer 渲染器
    const serverConfig = require('./webpack.server.config')
    const serverCompiler = webpack(serverConfig)
    const serverDevMiddleware = devMiddleware(serverCompiler, {
        logLevel: 'silent' // 关闭日志输出
    })

    serverCompiler.hooks.done.tap('server', () => {
        serverBundle = JSON.parse(
            serverDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-server-bundle.json'), 'utf-8')
        )
        update()
    })

    // 监视构建 clientManifest --> 调用 update --> 更新 Renderer 渲染器

    return onReady
}
```

## 客户端构建

按照上述将处理，对客户端生成 manifest 进行监听，变化后执行 update 方法。

```javascript
/**
* setup-dev-server.js
*/
const fs = require('fs')
const path = require('path')
const chokidar = require('chokidar')
const webpack = require('webpack')
const devMiddleware = require('webpack-dev-middleware')

const resolve = filePath => path.resolve(__dirname, filePath)

module.exports = (server, callback) => {
    let ready
    const onReady = new Promise(r => ready = r)

    // 监视构建 ==> 更新 Renderer，即调用 callback
    let template, serverBundle, clientManifest

    const update = () => {
        if (template && serverBundle && clientManifest) {
            ready()
            callback(serverBundle, template, clientManifest)
        }
    }

    update()
    // 监视构建 template --> 调用 update --> 更新 Renderer 渲染器
    const templatePath = resolve('../index.template.html')
    template = fs.readFileSync(templatePath, 'utf-8')
    // 监听文件变化的方法，此处使用第三方的 chokidar fs.watch  fa.watchFile chokidar
    chokidar.watch(templatePath).on('change', () => {
        update()
    })

    // 监视构建 serverBundle --> 调用 update --> 更新 Renderer 渲染器
    const serverConfig = require('./webpack.server.config')
    const serverCompiler = webpack(serverConfig)
    const serverDevMiddleware = devMiddleware(serverCompiler, {
        logLevel: 'silent' // 关闭日志输出
    })

    serverCompiler.hooks.done.tap('server', () => {
        serverBundle = JSON.parse(
            serverDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-server-bundle.json'), 'utf-8')
        )
        update()
    })

    // 监视构建 clientManifest --> 调用 update --> 更新 Renderer 渲染器
    const clientConfig = require('./webpack.client.config')
    const clientCompiler = webpack(clientConfig)
    const clientDevMiddleware = devMiddleware(clientCompiler, {
        publicPath: clientConfig.output.publicPath,
        logLevel: 'silent' // 关闭日志输出
    })

    clientCompiler.hooks.done.tap('client', () => {
        clientManifest = JSON.parse(
            clientDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-client-manifest.json'), 'utf-8')
        )
        update()
    })

    // 将 clientDevMiddleware 挂载到 Express 服务中，提供对其内部内存中数据的访问
    server.use(clientDevMiddleware)

    return onReady
}
```

render 需要传参，将回函数的 req 和 res 传入；

```javascript
...
server.get('*', isProd
    ? render
    : async (req, res) => {
        // 等待有了 Renderer 渲染器后，调用 render 进行渲染
        await onReady
        render(req, res)
    }
)
...
```

代码到达以上后，执行 ```yarn dev``` 启动开发模式，浏览器测试双向绑定的功能OK。

到目前为止，我们在开发模式下，修改代码已经能完成重新打包，但是还需要手动刷新浏览器，下边将介绍怎么自动刷新浏览器。

## 热更新

使用 webpack-hot-middleware 实现。```yarn add  webpack-hot-middleware -D``` 

```javascript
/**
* setup-dev-server.js
*/
const fs = require('fs')
const path = require('path')
const chokidar = require('chokidar')
const webpack = require('webpack')
const devMiddleware = require('webpack-dev-middleware')
const hotMiddleWare = require('webpack-hot-middleware')

const resolve = filePath => path.resolve(__dirname, filePath)

module.exports = (server, callback) => {
    let ready
    const onReady = new Promise(r => ready = r)

    // 监视构建 ==> 更新 Renderer，即调用 callback
    let template, serverBundle, clientManifest

    const update = () => {
        if (template && serverBundle && clientManifest) {
            ready()
            callback(serverBundle, template, clientManifest)
        }
    }

    update()
    // 监视构建 template --> 调用 update --> 更新 Renderer 渲染器
    const templatePath = resolve('../index.template.html')
    template = fs.readFileSync(templatePath, 'utf-8')
    // 监听文件变化的方法，此处使用第三方的 chokidar fs.watch  fa.watchFile chokidar
    chokidar.watch(templatePath).on('change', () => {
        update()
    })

    // 监视构建 serverBundle --> 调用 update --> 更新 Renderer 渲染器
    const serverConfig = require('./webpack.server.config')
    const serverCompiler = webpack(serverConfig)
    const serverDevMiddleware = devMiddleware(serverCompiler, {
        logLevel: 'silent' // 关闭日志输出
    })

    serverCompiler.hooks.done.tap('server', () => {
        serverBundle = JSON.parse(
            serverDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-server-bundle.json'), 'utf-8')
        )
        update()
    })

    // 监视构建 clientManifest --> 调用 update --> 更新 Renderer 渲染器
    const clientConfig = require('./webpack.client.config')
    clientConfig.plugins.push(new webpack.HotModuleReplacementPlugin())
    clientConfig.entry = [
        'webpack-hot-middleware/client?quiet=true&reload=true', // 和服务端交互处理热更新的一个脚本
        clientConfig.entry.app
    ]
    const clientCompiler = webpack(clientConfig)
    const clientDevMiddleware = devMiddleware(clientCompiler, {
        publicPath: clientConfig.output.publicPath,
        logLevel: 'silent' // 关闭日志输出
    })

    clientCompiler.hooks.done.tap('client', () => {
        clientManifest = JSON.parse(
            clientDevMiddleware.fileSystem.readFileSync(resolve('../dist/vue-ssr-client-manifest.json'), 'utf-8')
        )
        update()
    })

    server.use(hotMiddleWare(clientCompiler, {
        log: false // 关闭日志输出
    }))

    // 将 clientDevMiddleware 挂载到 Express 服务中，提供对其内部内存中数据的访问
    server.use(clientDevMiddleware)

    return onReady
}
```

# 编写通用应用注意事项

[https://ssr.vuejs.org/zh/guide/universal.html#%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84%E6%95%B0%E6%8D%AE%E5%93%8D%E5%BA%94](https://ssr.vuejs.org/zh/guide/universal.html#%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84%E6%95%B0%E6%8D%AE%E5%93%8D%E5%BA%94)

# 路由处理

## 配置 VueRouter

安装 ```yarn add vue-router```;

src 目录下创建 pages 目录用于存放页面组件，创建几个页面用于路由测试；

src 目录下创建 router 目录，下边创建 index.js 配置路由；

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from "../pages/Home"

Vue.use(VueRouter)

export const createRouter = () => {
    const router = new VueRouter({
        mode: 'history', // 兼容前后端
        routes: [
            {
                path: '/',
                name: 'home',
                component: Home
            },
            {
                path: '/about',
                name: 'about',
                component: () => import('@/pages/about')
            },
            {
                path: '*',
                name: 'error404',
                component: () => import('@/pages/404')
            }
        ]
    })
    return router
}
```

将路由注册到根实例

```javascript
/**
 * app.js
 * 通用的启动入口
 *
 * app.js 是我们应用程序的「通用 entry」。在纯客户端应用程序中，我们将在此文件中创建根 Vue 实例，
 * 并直接挂载到 DOM。但是，对于服务器端渲染(SSR)，责任转移到纯客户端 entry 文件。app.js 简单地使用 export 导出一个 createApp 函数：
 */
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from "@/router"

// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp () {
    const router = createRouter()
    const app = new Vue({
        router, // 把路由挂载待 Vue 根实例
        render: h => h(App) // 根实例简单的渲染应用程序组件。
    })
    return { app, router }
}

```

## 适配服务端入口

官方文档的 [路由和代码分割](https://ssr.vuejs.org/zh/guide/routing.html#%E4%BD%BF%E7%94%A8-vue-router-%E7%9A%84%E8%B7%AF%E7%94%B1) 提供了将路由适配到服务端入口中的方法。参考官方文档，本地将其内容改为使用 async 和 await 方式。

```javascript
/**
 * entry-server.js
 * 服务器 entry 使用 default export 导出函数，并在每次渲染中重复调用此函数。
 * 此时，除了创建和返回应用程序实例之外，它不会做太多事情 - 但是稍后我们将在此执行服务器端路由匹配 (server-side route    	matching) 和数据预取逻辑 (data pre-fetching logic)。
 */
import { createApp } from './app'

export default async context => {
    // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
    // 以便服务器能够等待所有的内容在渲染前，
    // 就已经准备就绪。
    const { app, router } = createApp()

    // 设置服务器端 router 的位置
    router.push(context.url)

    // 等到 router 将可能的异步组件和钩子函数解析完
    new Promise(router.onReady.bind(router))
    return app
}
```

## 服务端 server 适配

```javascript
// server.js
...
const render = async (req, res) => {
    try {
        const html = await renderer.renderToString({
            title: "vue-ssr",
            meta: `<meta name="description" content="我是 vue ssr">`,
            url: req.url
        })

        // res.setHeader('Content-Type', 'text/html; charset=utf8')
        res.end(html)
    } catch (e) {
        res.status(500).end('Internal Server Error.')
    }
}
...
```

## 适配客户端入口

官方文档的 [路由和代码分割](https://ssr.vuejs.org/zh/guide/routing.html#%E4%BD%BF%E7%94%A8-vue-router-%E7%9A%84%E8%B7%AF%E7%94%B1) 提供了将路由适配到客户端入口中的方法。

需要在挂载 app 之前调用 `router.onReady`，因为路由器必须要提前解析路由配置中的异步组件，才能正确地调用组件中可能存在的路由钩子。

```javascript
/**
 * 客户端 entry 只需创建应用程序，并且将其挂载到 DOM 中
 */
import { createApp } from './app'

// 客户端特定引导逻辑……

const { app, router } = createApp()

router.onReady(() => {
    app.$mount('#app')
})
```

执行 ```yarn dev``` 启动服务，打开浏览器 [http://localhost:3000](http://localhost:3000) ，测试路由跳转 OK；

![路由测试-Home页](2021-07-01_144325.png)

![路由测试-About页](2021-07-01_144429.png)

![路由测试-404页](2021-07-01_144524.png)

# 管理页面的 Head 内容

目前项目中的 index.template.html 作为页面模板，其中的 Head 内容是共享的，即只有一份，不管访问哪个页面，内容都一样。

我们希望各页面能自己设置自己的标题 Title 和 原数据 Meta 等信息，可以参考 [https://ssr.vuejs.org/zh/guide/head.html](https://ssr.vuejs.org/zh/guide/head.html) ，也可以使用其他的三方解决方案，比如下面我们就使用 [**vue-meta**](https://vue-meta.nuxtjs.org/) 来管理各个页面的头部信息。

```yarn add vue-meta```

## 配置 vue-meta

app.js 通用入口文件中，使用 Vue 配置 vue-meta，并 mixmin 一个 title 的模板；

```javascript
...
import VueMeta from 'vue-meta'
Vue.use(VueMeta)
Vue.mixin({
    metaInfo: {
        titleTemplate: '%s - Vue-SSR'
    }
})
...
```

entry-server.js 中在上下文上挂载 meta 属性，保证页面模板中能动态获取。

```javascript
...
    const meta = app.$meta()
    // 设置服务器端 router 的位置
    router.push(context.url)
    context.meta = meta
...
```

index.template.html 模板中获取 meta 配置。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    {{{ meta.inject().title.text() }}}
    {{{ meta.inject().meta.text() }}}
</head>
<body>
<!--vue-ssr-outlet-->
</body>
</html>
```

## vue-meta 具体使用

此处示例给 Home 页和 About 页设置自己的 Title。

```vue
<!--Home.vue-->
...
<script>
    export default {
        name: "Home",
        metaInfo: {
            title: '首页'
        }
    }
</script>
...
<!--About.vue-->
...
<script>
    export default {
        name: "About",
        metaInfo: {
            title: '关于'
        }
    }
</script>
...
```

.vue 对应页面设置的 metaInfo.title 就会替换到 app.js 中 mixin 的 titleTemplate 中的 "%s" 位置，并解析为 title 插入到模板的 meta.inject().title.text() 位置。

![vue-meta-pic1](2021-07-02_100217.png)

![vue-meta-pic2](2021-07-02_100330.png)

# 数据预取和状态管理

在服务器端渲染(SSR)期间，我们本质上是在渲染我们应用程序的"快照"，所以如果应用程序依赖于一些异步数据，**那么在开始渲染过程之前，需要先预取和解析好这些数据**。

在客户端，在挂载 (mount) 到客户端应用程序之前，需要获取到与服务器端应用程序完全相同的数据 - 否则，客户端应用程序会因为使用与服务器端应用程序不同的状态，然后导致混合失败。

为了解决这个问题，获取的数据需要位于视图组件之外，即放置在专门的数据预取存储容器(data store)或"状态容器(state container)"中。首先，在服务器端，我们可以在渲染之前预取数据，并将数据填充到 store 中。此外，我们将在 HTML 中序列化(serialize)和内联预置(inline)状态。这样，在挂载(mount)到客户端应用程序之前，可以直接从 store 获取到内联预置(inline)状态。

即我们可以使用官方的状态管理库 Vuex。

[https://ssr.vuejs.org/zh/guide/data.html](https://ssr.vuejs.org/zh/guide/data.html) 

## 基于 Vuex 创建容器

> ```yarn add vuex``` 
>
> src 下创建 store 目录，store 下创建 index.js
>
> 配置 index.js

```javascript
// store/index.js
import Vuex from 'vuex'
import Vue from 'vue'
import axios from "axios"

Vue.use(Vuex)

export const createStore = () => {
    return new Vuex.Store({
        state: () => {
            return {
                posts: []
            }
        },
        mutations: {
            setPosts: (state, payload) => {
                state.posts = payload
            }
        },
        actions: {
            // 在服务端渲染期间务必让 action 返回一个 Promise
            async getPosts({ commit }) {
                const { data } = await axios.get('https://cnodejs.org/api/v1/topics')
                commit('setPosts', data.data)
            }
        }
    })
}
```

## app.js 中将 store 挂载到 Vue 根实例

```javascript
/**
 * 通用的启动入口
 *
 * app.js 是我们应用程序的「通用 entry」。在纯客户端应用程序中，我们将在此文件中创建根 Vue 实例，
 * 并直接挂载到 DOM。但是，对于服务器端渲染(SSR)，责任转移到纯客户端 entry 文件。app.js 简单地使用 export 导出一个 createApp 函数：
 */
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from "@/router"
import VueMeta from 'vue-meta'
import { createStore } from "@/store"

Vue.use(VueMeta)
Vue.mixin({
    metaInfo: {
        titleTemplate: '%s - Vue-SSR'
    }
})

// 导出一个工厂函数，用于创建新的
// 应用程序、router 和 store 实例
export function createApp () {
    const router = createRouter()
    const store = createStore()
    const app = new Vue({
        router, // 把路由挂载待 Vue 根实例
        store, // 把容器挂载到 Vue 根实例
        render: h => h(App) // 根实例简单的渲染应用程序组件。
    })
    return { app, router, store }
}
```

## 数据预取处理页面

```vue
<template>
    <div>
        <h1>Posts List</h1>
        <ul>
            <li v-for="post in posts" :key="post.id">
                {{ post.title }}
            </li>
        </ul>
    </div>
</template>
<script>
    import axios from 'axios'
    import { mapState, mapActions } from 'vuex'
    export default {
        name: "Posts",
        data() {
            return {
                // posts: []
            }
        },
        computed: {
            ...mapState(['posts'])
        },
        // Vue SSR 特殊的为服务端渲染提供的生命周期钩子函数
        serverPrefetch() {
            // 发起 action 返回 promise
            return this.getPosts()
        },
        methods: {
          ...mapActions(['getPosts'])
        },
        // 服务端渲染只支持 beforeCreate 和 created 钩子函数
        // 且服务端渲染不会等待它们中的异步操作
        // 不支持响应式数据
        // 以下方法在服务端渲染中不会工作，实际是在客户端渲染出来的
        // async created() {
        //     const { data } = await axios({
        //         method: 'GET',
        //         url: 'https://cnodejs.org/api/v1/topics'
        //     })
        //
        //     this.posts = data.data
        // }
    }
</script>
<style scoped>
</style>
```

## entry-server.js

```javascript
/**
 * 服务器 entry 使用 default export 导出函数，并在每次渲染中重复调用此函数。
 * 此时，除了创建和返回应用程序实例之外，它不会做太多事情 - 但是稍后我们将在此执行服务器端路由匹配 (server-side route matching) 和数据预取逻辑 (data pre-fetching logic)。
 */
import { createApp } from './app'

export default async context => {
    // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
    // 以便服务器能够等待所有的内容在渲染前，
    // 就已经准备就绪。
    const { app, router, store } = createApp()

    const meta = app.$meta()

    // 设置服务器端 router 的位置
    router.push(context.url)

    context.meta = meta

    // 等到 router 将可能的异步组件和钩子函数解析完
    await new Promise(router.onReady.bind(router))

    // 当服务端渲染结束后调用
    context.rendered = () => {
        // Renderer 会把 context.state 数据对象内联到页面模板中
        // 最终发送给客户端的页面中会包含一段脚本：window.__INITIAL_STATE__ = context.state
        // 客户端就要把页面中的 window.__INITIAL_STATE__ 拿出来填充到客户端 store 容器中
        context.state = store.state // 将服务端状态挂载到客户端状态上，保持一致，才能在客户端渲染
    }
    return app
}
```

## entry-client.js

```javascript
/**
 * 客户端 entry 只需创建应用程序，并且将其挂载到 DOM 中
 */
import { createApp } from './app'

// 客户端特定引导逻辑……

const { app, router, store } = createApp()
// 客户端将服务端设置到 widow 的状态配置到客户端
if (window.__INITIAL_STATE__) {
    store.replaceState(window.__INITIAL_STATE__)
}

router.onReady(() => {
    app.$mount('#app')
})
```

重启应用后，点击 Posts 路由，即可得到 posts 服务端直接渲染的页面；

![数据预取和状态管理](2021-07-02_140837.png)

# 相关链接：

[https://ssr.vuejs.org/](https://ssr.vuejs.org/);  

[https://vue-meta.nuxtjs.org/](https://vue-meta.nuxtjs.org/);

[https://github.com/paulmillr/chokidar](https://github.com/paulmillr/chokidar);

[https://github.com/webpack/webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware);



本文完整 Demo 地址：[https://github.com/SeaChan0117/vue-SSR](https://github.com/SeaChan0117/vue-SSR) ；



