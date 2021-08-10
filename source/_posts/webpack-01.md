---
title: webpack-01
date: 2021-08-10 10:34:09
categories:
- 大前端
- 工程化
tags:
- webpack
---
![gulp](bg.jpg)

随着 ES6 新特性模块化在语言层面的实现，模块化的一些问题需要被解决：

- ES Modules 存在环境兼容问题；
- 模块文件过多，网络请求频繁；
- 所有的前端资源都需要模块化；

所以为了解决以上问题，就需要有一个能编译、打包的工具，处理相关兼容性，能将多文件打包到一起，减少文件请求，还要支持其他一些静态资源处理，是针对整个前端项目来讲的工具。所以市面上应运而生了几个主流的打包工具，**Webpack**，**Rollup**，**Parcel**，本文重点学习记录最受欢迎的 **Webpack**。
<!--more-->

# Webpack 快速上手

初始化项目 **package.json**，`yarn init --yes`；

安装 **Webpack **以及它的 **cli **模块，`yarn add webpack webpack-cli --dev`；

![](01.jpg)

执行 `yarn webpack` 后，会自动根据 **src** 下的 **index.js** 作为入口文件进行打包，并生成打包文件夹dist，其中打包文件为 **main.js**；

![](02.jpg)

也可以将命令定义为 **npm scripts** 命令中，减少命令输入操作，执行 `yarn build` 即可；

![](03.jpg)

以上是 **webpack4** 之后支持的零配置打包，打包回默认从 **‘src/index.js’** 文件作为入口，打包到 **‘dist/main.js’**。

# Webpack 配置文件

大多时候我们需要自定义入口文件或添加一些其他的配置参数，这时我们就需要一个配置 **webpack** 配置文件，在项目根目录创建其配置文件 **webpack.config.js** ；

```js
const path = require('path')

module.exports = {
    entry: './src/main.js',     // 入口文件，相对路径时，'./'不能省略
    output: {                   // 指定输入文件路径
        filename: 'bundle.js',  // 输入文件名称
        path: path.join(__dirname, 'output')          // 输出文件路径，必须是绝对路径
    }
}
```

![](04.jpg)

# Webpack 工作模式

**webpack** 有针对工作模式的用法，相当于是么针对几组工作环境预设的配置，大大简化了配置的复杂程度，有 **production**，**development** 和 **none** 模式，默认是 **production** 模式，

- 生产模式 `yarn webpack --mode production`；
- 开发模式 `yarn webpack --mode development`；
- none模式 `yarn webpack --mode none`；

关于各个模式的差异可以[参考官方文档](https://link.zhihu.com/?target=https%3A//webpack.js.org/configuration/mode/)进行查看；

[Mode | webpack](https://link.zhihu.com/?target=https%3A//webpack.js.org/configuration/mode/)

也可以通过配置文件配置 **mode** 属性指定对应的工作模式；

```js
const path = require('path')

module.exports = {
    mode: 'development',        // 工作模式 production | development | none
    entry: './src/main.js',     // 入口文件，相对路径时，'./'不能省略
    output: {                   // 指定输入文件路径
        filename: 'bundle.js',  // 输入文件名称
        path: path.join(__dirname, 'output')          // 输出文件路径，必须是绝对路径
    }
}
```

# Webpack 资源模块加载

**src** 下添加 **main.css** 文件，修改 **webpack** 打包入口为 **main.css** ，打包报错，原因是 **webpack** 打包默认是找 **js** 文件，其他文件需要对应的加载器 **loader** ；

安装 **css-loader**：`yarn add css-loader style-loader --dev`；

```js
const path = require('path')

module.exports = {
    mode: 'none',        // 工作模式 production | development | none
    entry: './src/main.css',     // 入口文件，相对路径时，'./'不能省略
    output: {                   // 指定输入文件路径
        filename: 'bundle.js',  // 输入文件名称
        path: path.join(__dirname, 'dist')          // 输出文件路径，必须是绝对路径
    },
    module: {
        rules: [                        // rules 针对其他资源加载规则的配置项
            {
                test: /.css$/,          // 匹配打包过程中遇到的文件路径
                use: [                  // 指定匹配到的文件要使用的loader
                    'style-loader',
                    'css-loader'
                ]
            }
        ]
    }
}
```

**Loader** 是 **Webpack** 实现整个前端模块化的核心，通过不同的 **loader** 可以加载任何类型的资源。

# Webpack 导入模块资源

因为前端是以 **JavaScript** 驱动的，所以一般都是以 **JavaScript** 作为打包入口，通过 **import** 的方式在 **js** 中引入 **css** 文件，不管何种资源，在编码的时候根据代码的需要动态导入资源；

# Webpack 文件资源加载器

加载图片等文件资源时要使用文件资源加载器，`yarn add file-loader --dev` ；

![](05.jpg)

此时如果是 **webpack4+** 的版本，代码中使用引入的文件会报找不到图片引用，需要在 **output** 中配置 **publicPath** 属性为 **‘dist/’** ，**webpack5+** 会找服务器下打包文件的根路径，不会报错；

![webpack5+](06.jpg)

![](07.jpg)

![配置publicPath后](08.jpg)

文件加载器打包过程，**webpack** 在打包时，遇到了图片文件，根据配置文件中的配置匹配到对应的文件加载器，文件加载器开始工作，先将导入的文件拷贝到输出目录，然后再将文件拷贝到输出目录的路径，作为当前这个模块的返回值返回，这样针对应用来说，所需要的资源就被发布出来了，同时也可以通过这个模块的导出成员拿到这个文件的导出路径。

![](09.jpg)

# Webpack Data URLs 与 url-loader

除了 **file-loader**，我们还可以使用 **Data URLs** 去表示一个文件，它的当前 **url** 文本就包含了文件内容，使用这种 **url** 时就不会再去发送**http** 请求。

详见：[Data URLs - HTTP](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)

在 **webpack** 打包时也可以使用这种格式去表示任何格式的文件，使用专门的加载器 **url-loader**，`yarn add url-loader --dev`，然后将之前配置文件中的 **file-loader** 改为 **url-loader** ，重新打包查看页面上的图片路径。

![](10.jpg)

```js
...
{
    test: /.png$/,
    use: {
        loader: 'url-loader',       // 适合体积较小的资源，大小小于10KB？偏大的会交给使用file-loader，注意一定要同时安装file-loader
        options: {
            limit: 10 * 1024
        }
    }
}
...
```

# Webpack 常用加载器分类

资源加载器主要是用于处理在打包过程中遇到的资源文件，常用的 **loader** 分为以下几类；

- 编译转换类，把编译模块转换为 **JavaScript** 代码；
- 文件操作类，通常是把加载的资源模块拷贝到输出目录，并将文件的路径向外导出；
- 代码检查类，加载的资源是代码时，可以通过其对代码进行校验，统一代码风格，维护代码质量；

# Webpack 与 ES6

由于 **webpack** 只是打包工具，并不会我们转换 **ES6** 新特性，所以打包编译过程需要安装配置对应的 **loader** 去对 **ES6** 代码进行转换，`yarn add babel-loader @babel/core @babel/preset-env --dev`；

```js
{
    test: /.js$/,
    use: {
        loader: 'babel-loader',         // babel转换
        options: {
            presets: ['@babel/preset-env']       // 匹配新特性，全部转换
        }
    }
}
```

# Webpack 加载资源的方式

**webpack** 加载 **JavaScript** 模块的方式有以下几种：

- ESM 中 import；
- commonJS 中的 require；
- 还有 AMD 标准下的 define 和 require；

其他的非 **JavaScript** 加载也能触发相应的资源加载，如 **CSS** 中使用的 **@import **指令和 **url** 路径，**HTML **代码中图片标签的 **src** 属性，也能触发相应的资源加载；

**HTML** 的加载需要配置对应的 **loader**，`yarn add html-loader --dev`；

```js
{
    test: /.html$/,
    use: {
        loader: 'html-loader',      // html 打包对应的loader
        options: {
            attrs: ['img:src', 'a:href']        // attrs 指定额外需要处理的属性，对这些属性也触发资源加载，默认已有 img:src
        }
    }
}
```

# Webpack 核心工作原理

在项目中一般都是散落着各种各样的代码及资源文件，**webpack** 会根据我们的配置找到一个文件作为打包入口（一般是一个 .js 文件），然后顺着入口文件中的代码，其中的 **import**、**require** 之类的语句解析推断出所依赖的资源模块，然后分别去解析每一个模块对应的依赖，最后就形成了整个项目中用到文件之间的一个依赖关系的依赖树，然后 **webpack** 会递归遍历这个依赖树，找到每个节点所依赖的资源文件，再根据配置文件中的 **rules** 属性找到这个模块对应的资源加载器，然后交给对应的加载器去加载这个模块，最后将加载后的结果放到打包结果中，实现整个项目的打包。其中 **Loader** 机制是 **Webpack** 的核心。

# Webpack 插件机制

为了增强 **Webpack** 在自动化方面的能力，引入了 **Plugin** 解决处理打包以外其他的自动化工作，例如，在打包前自动清除 **dist** 目录，拷贝不需要打包的静态资源到目标目录，或者压缩打包后的代码等。

# Webpack 自动清除输出目录插件

`yarn add clean-webpack-plugin --dev`；

```js
const {CleanWebpackPlugin} = require('clean-webpack-plugin')
...
plugins: [                          // 插件集合
    new CleanWebpackPlugin()
]
...
```

# Webpackc自动生成HTML插件

如果项目中默认有硬编码的 **index.html** 文件作为页面入口，就会存在问题：

- 打包时如果依赖的 js 文件路径发生变化，需要手动修改 script 引入路径；
- 打包部署时需要同时替换打包文件和 index.html 文件;

我们可以通过 **Webpack** 自动生成 **html** 文件，**html** 文件也生成到 **dist** 目录，其中的依赖也是通过 **webpack** 打包依赖动态注入的，不需要手动修改，能确保路径引用的正确。

`yarn add html-webpack-plugin --dev`，并在配置文件的 **plugins** 集合添加相应的实例对象，删除 **publicPath** 属性，保证生成的 **html** 对 **js** 的引用正确；

![](11.jpg)

再次打包，**dist** 下自动生成 **html** 文件并正确引入打包后的 **js** 文件，bingo！

还可以对插件进行相关配置项传入，修改生成的 **html** 内容；

![](12.jpg)

其他详细配置请参考 **[html-webpack-plugin](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/html-webpack-plugin)** 中的配置表。

如果添加多个 **HtmlWebpackPlugin** 实例对象就可生成多个 **html** 文件；

![](13.jpg)

# Webpack 静态文件资源拷贝插件

项目中一些文件在打包时只需要做拷贝处理，这是可以使用相应的插件处理，`yarn add copy-webpack-plugin --dev`；

![](14.jpg)

相关配置请参考 **[copy-webpack-plugin](https://link.zhihu.com/?target=https%3A//www.npmjs.com/package/copy-webpack-plugin)**

以上，我们了解并尝试使用了 **webpack** 的一些基本插件，在实际应用中，我们可以根据自己的需要去查找对应的插件并使用即可。

其中loader是打包的核心机制，**webpack** 将一切文件视为模块，但是 **webpack** 原生是只能解析 **js** 文件，如果想将其他文件也打包的话，就会用到 **loader**， 所以 **Loader** 的作用是让 **webpack** 拥有了解析非 **JavaScript** 文件的能力；**plugin** 可以扩展 **webpack** 的功能，让 **webpack** 更加灵活。 在 **webpack** 运行的生命周期中有很多的钩子函数，**plugin** 挂载到对应的钩子，通过 **webpack** 提供的 **API** 去修改输出结果，**plugin** 可以作用贯穿在整个编译阶段，功能更加丰富，完成 **loader** 不能做到的工作，比如代码优化，打包压缩，自定生成文件等，功能强大到可以用来处理各种各样的任务。

