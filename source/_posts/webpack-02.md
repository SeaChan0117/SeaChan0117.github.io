---
title: Webpack开发体验问题
date: 2021-08-10 11:28:00
categories:
- 大前端
- 工程化
tags:
- webpack
---
![gulp](bg.jpg)

在上一篇中讲了 **Webpack** 的基础学习使用，但实际开发过程中我们需要一个更灵活的开发环境，或某些问题可以通过 **Webpack** 来解决，本文将列举一些常见的问题或示例去介绍怎样增强 **Webpack** 的开发体验。

# 自动编译

使用 **webpack-cli** 的 **watch** 模式，该模式能监听文件变化，完成自动重新打包，使用比较简单，只需要在启动时添加 **watch** 参数即可，`yarn webpack --watch`，此时会启动监听模式，并且不会立即退出命令窗口，会等待文件变化，并再次完成打包工作，直到我们手动结束该模式；

# 自动刷新浏览器

我们在开发时已经完成了自动编译，如果能自动刷新浏览器，我们的工作将更加顺滑，不用每次修改代码后手动刷新浏览器，**webpack dev server** 集成了自动编译和自动刷新浏览器的功能，使用它可以方便解决自动编译和自定刷新的问题，`yarn add webpack-dev-server --dev`；安装完成执行 `yarn webpack serve`（**webpack4+** 使用 `yarn webpack-dev-server`，也可定义到 **npm scripts** 中减少操作），就能自动打包并启动一个服务去运行打包结果，访问启动的本地服务即可（也可添加--open参数自动打开浏览器），且修改代码后，能自动编译并刷新浏览器，这个过程的编译不会生成dist目录，它是在内存当中完成的，减少了磁盘操作，提高了构建的效率；

![](01.jpg)

**webpack dev server** 会将默认的打包文件作为开发服务器的资源文件，只要能通过 **webpack** 打包输出的文件，都可以被访问到，如果其他一些静态资源也需要被开发服务器访问，就需要额外的告知 **webpack dev server**，这时需要有相应的配置去处理，添加相应配置；

![](02.jpg)

当拥有单独的 API 后端开发服务器并且希望在同一域上发送 API 请求时，**webpack dev server** 还支持配置代理 API 服务，更多详细配置参考 [开发服务器(DevServer) | webpack 中文文档](https://link.zhihu.com/?target=https%3A//webpack.docschina.org/configuration/dev-server/%23devserverproxy)

![](03.jpg)

# Source Map 配置

在配置中添加 **devtool** 属性，用于配置与开发相关的一些辅助工具，此时将其配置成 **source-map**，打包后生成相关的 **source map**，运行后，相关的代码执行能直接定位到我们的源代码；

![](04.jpg)

![浏览器打印定位在源代码](05.jpg)

**webpack** 对 **source map** 的支持在最新的版本中有二十多种模式，我们需要选择自己需要的去使用，以到达高效准确处理问题，详情请参考 [Devtool | webpack](https://link.zhihu.com/?target=https%3A//webpack.js.org/configuration/devtool/%23devtool)

相关选择项参考：

- eval-是否使用eval执行代码块；
- cheap-Source map是否包含行信息；
- module-是否能够得到loader处理之前的源代码；

开发环境下 **cheap-module-eval-source-map** 比较适合使用框架的项目，如果一行代码不是很长，这个模式能容易帮忙找到错误信息位置，代码经过 **loader** 转换后差异较大，这个模式能定位源码位置，且该模式重写打包速度相对较快；

生产打包时处于安全考虑，建议使用 **none** 或者 **nosources-source-map**，这样不会把源代码暴露在生产环境。

# HMR（Hot Module Replacement）配置及使用

模块热替换能能更大程度的提高我们的开发效率，在程序运行时，修改某个模块的数据，能在不刷新浏览器的情况下实时同步到页面，而保留之前页面上的输入数据或缓存。

可以通过命令 **webpack-dev-server --hot** 开启，也可在配置文件中配置，这时需要一个 **webpack** 的插件，配置使用如下，此时启动本地服务，尝试修改样式文件后，浏览器未刷新，但是浏览器上对应的样式已对应生效。

![](06.jpg)

修改 **js** 时发现浏览器还是会刷新，**Webpack** 的 **HMR** 实际需要我们手动通过代码去处理热替换的逻辑，样式文件能生效是因为 **style-loader** 已经帮我们处理过了，而像 **js** 模块这种，是不确定的，没有规律的，所以没有通用的 **HMR** 方案，只能自己手动去处理。

![HMR处理js热替换简单示例](07.jpg)

![图片热替换示例](08.jpg)

# 不同环境配置

可以根据环境配置不同的配置文件，生成不同的包，在相应的环境中使用，方法一般有以下两种：

- 配置文件根据环境不同导出不同的配置；
- 一个环境对应一个配置文件；

对于简单或小型项目而言，在同一个配置文件中进行相应配置会比较方便，第一种配置简单示例，主要是判断传入参数是否是**production**，执行命令 **webpack --env production** 进行构建时传入；

![](09.jpg)

如果项目较大，在同一个文件中进行配置会太繁杂不够清晰，所以建议使用第二种方法，根据不同环境配置不同的配置文件，一般有一个公共的配置文件作为配置项基础，其他各个环境对应自己的配置，使用时，将基础配置导入，再根据自己环境相关进行配置后导出，以下是一个简单示例，打包时添加 **--config** 指定对应的 **js** 配置文件即可；

![](10.jpg)

![](11.jpg)

# DefinePlugin

**DefinePlugin** 允许在编译时创建配置的全局常量，这在需要区分开发模式与生产模式进行不同的操作时，非常有用。例如，如果想在开发构建中进行日志记录，而不在生产构建中进行，就可以定义一个全局常量去判断是否记录日志。设置它，就可以忘记开发环境和生产环境构建的规则。

传递给 **DefinePlugin** 的每个键都是一个标识符或多个以.连接的标识符。

- 如果该值为字符串，它将被作为代码片段来使用。
- 如果该值不是字符串，则将被转换成字符串（包括函数方法）。
- 如果值是一个对象，则它所有的键将使用相同方法定义。
- 如果键添加 **typeof** 作为前缀，它会被定义为 **typeof** 调用。

这些值将内联到代码中，从而允许通过代码压缩来删除冗余的条件判断。详细使用请参考[DefinePlugin | webpack 中文文档](https://link.zhihu.com/?target=https%3A//webpack.docschina.org/plugins/define-plugin/)

![](12.jpg)

![DefinePlugin示例结果](13.jpg)

# Tree Shaking

**Tree Shaking** 是一个术语，通常用于描述移除 **JavaScript **上下文中的未引用代码(dead-code)。**Webpack** 在 **production** 模式下自动开启 **TreeShaking**，检索代码中未引入的代码并处理。其他模式想要使用 **TreeShaking**，可以使用配置项[optimization](https://link.zhihu.com/?target=https%3A//webpack.docschina.org/configuration/optimization/)去实现。

使用 **TreeShaking** 的一些前提：

- 使用 ES2015 模块语法（即 import 和 export）。
- 确保没有编译器将您的 ES2015 模块语法转换为 CommonJS 的（顺带一提，这是现在常用的 @babel/preset-env 的默认行为，详细信息请参阅[文档](https://link.zhihu.com/?target=https%3A//babeljs.io/docs/en/babel-preset-env%23modules)）。
- 在项目的 package.json 文件中，添加 "sideEffects" 属性。
- 使用 mode 为 "production" 的配置项以启用[更多优化项](https://link.zhihu.com/?target=https%3A//webpack.docschina.org/concepts/mode/%23usage)。

# 代码分割

随着项目代码越来越多，最终打包的 **bundle** 会很大，会造成资源太大，影响加载，而且并不是所有的代码都是一开始就需要加载的，这个时候就需要对代码进行分包处理，打包成多个 **bundle**，我们可以根据多入口文件进行打包，或者使用动态导入的方式进行打包，这样都能生成多个 **bundle**；

多入口时将配置文件的 **entry** 设置为对象，一个属性对应一个入口，输入的 **filename** 需要使用 **[name]** 占位符，保证在输出和输入时一一对应，自动生成 **html** 时要使用 **chunks** 属性指定 **html** 对应的依赖 **js** 包，否则会默认引入所有 **js** ；

![](14.jpg)

注意，此时可以配置 **splitChunks** 属性将公共模块打包到一起：

![](15.jpg)

动态导入即按需加载，需要用到某个模块时再去加载这个模块，能极大程度的较少带宽和流量。**webpack** 中所有动态导入的模块都会被提取到单独的模块自动分包，通过代码逻辑去控制是否需要加载某个模块即可，即在需要使用某个模块时，使用 **import()** 动态导入模块即可；此时打包出来的文件名不是我们想要的和原文件名相关的，可以使用魔法注释去处理。

![](16.jpg)

# MiniCssExtractPlugin

**MiniCssExtractPlugin** 插件能将 **css** 提取成单个模块，即实现 **css** 的按需加载，先安装 `yarn add mini-css-extract-plugin --dev`，配置后构建，生成相应的 **css**，注意，这里最好是当 **css** 文件超过某个大小（建议150KB），再使用此方法，文件太小增加了资源请求次数，对于优化来说适得其反。

![](17.jpg)

此时打包的 **css** 文件内容是未经压缩的，我们可以使用 **webpack** 官方推荐的压缩 **css** 插件进行处理，`yarn add optimize-css-assets-webpack-plugin --dev`，并配置对应插件，构建后查看生成的 **css** 文件，发现代码已压缩。

![](18.jpg)

此时还有一个问题，将 **OptimizeCssAssetsWebpackPlugin** 插件放在 **plugins** 中的话，无论什么时候 **css** 都会被压缩，这里应该放在 **optimization** 的 **minimizer** 下，使用 **OptimizeCssAssetsWebpackPlugin** 后，原本能自动压缩的 **js** 文件不能自动压缩了，原因是 **minimizer** 配置项会让 **webpack** 认为开发者要自定义配置压缩项，此时默认的压缩会被覆盖，我们需要手动添加 **js** 的压缩插件并使用，`yarn add terser-webpack-plugin --dev`。

![](19.jpg)

# 输出文件名hash

在前端开发过程中，一般都会最大限度去利用缓存，在实际应用中，即使服务器设置了缓存策略，可能构建的项目无法实现静态资源缓存，**webpack** 提供了文件名hash的方式去处理这个问题，**webpack** 对打包文件名支持三种hash；

**hash：**

**hash** 是工程级别的，即每次修改任何一个文件，所有文件名的hash值都将改变。所以一旦修改了任何一个文件，整个项目的文件缓存都将失效。

![](20.jpg)

**chunkhash：**

**chunkhash** 根据不同的入口文件(Entry)进行依赖文件解析、构建对应的chunk，生成对应的哈希值，一路 **chunk** 的 **hash** 值相同。

![](21.jpg)

**contenthash：**

**contenthash** 是针对文件内容级别的，只有你自己模块的内容变了，那么 **hash** 值才改变。

![](22.jpg)

我们还可以指定 **hash** 的长度，如下图指定了8位长度的 **contenthash**：

![](23.jpg)



以上是一些 **webpack** 基本的使用配置，基于 **nodeJS** 的发展，前端的能力或者说生态越来越好，**webpack** 又是一大神器，要使用好它还需要在更多的实战中去经历和使用，更多详细内容请参考[官方文档](https://link.zhihu.com/?target=https%3A//webpack.docschina.org/concepts/)。
