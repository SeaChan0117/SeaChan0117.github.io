---
title: Gulp自动化构建工作流封装
date: 2021-08-10 09:23:06
categories:
- 大前端
- 工程化
tags:
- gulp
cover: https://tse1-mm.cn.bing.net/th/id/R-C.a04c6d4902fe140cf7df7207212e1f4d?rik=DTzKlZiCsyHl1w&riu=http%3a%2f%2fwww.michaellerner.net%2fwp-content%2fuploads%2f2016%2f09%2fgulp-logo.jpg&ehk=S79Nwe6ktUUU00%2bq29NiHTeBz4HolYy5q8oZqO34Rho%3d&risl=&pid=ImgRaw&r=0
---
![gulp](bg.jpg)

之前咱们通过实际案例深入 `gulp` 的学习使用，但是 `gulpfile.js` 的复用只能当做一个模板在需要时拷贝到项目中，或者作为代码段统一在相应的项目中使用，但是如果涉及到相应gulp插件升级或者有问题需要修复时，所以的项目都要统一处理一遍，不利于后限整体维护，接下来就需要我们提取一个可复用的自动化构建工作流。

`gulpfile` + `gulp` = 自动化工作流；

我们需要去创建一个模块，然后将它发布到 `npm`，最后在我们的项目中去使用这个模块即可；

我这在 `git` 上创建了一空项目用于这次学习，最后将会成为打包模块去使用，`clone` 到本地初始化项目目录大体如下；
<!--more-->

![](01.jpg)

# 提取 gulpfile 到模块

先将 `gulpfile.js` 文件的内容全部移入模块的 `index.js` 中；

![](02.jpg)

将原项目（`cs-gulp-demo`）开发依赖移入模块（`cs-pages`）依赖，因为最后我们的模块是需要工作的，工作的时候是需要 `dependencies` 下的依赖，所以这里不是拷贝到 `devDependencies` ，拷贝完成后执行 `yarn` 安装依赖；

```json
"dependencies": {
    "@babel/core": "^7.13.13",
    "@babel/preset-env": "^7.13.12",
    "browser-sync": "^2.26.14",
    "del": "^6.0.0",
    "gulp": "^4.0.2",
    "gulp-babel": "^8.0.0",
    "gulp-clean-css": "^4.3.0",
    "gulp-htmlmin": "^5.0.1",
    "gulp-if": "^3.0.0",
    "gulp-imagemin": "^7.1.0",
    "gulp-load-plugins": "^2.0.6",
    "gulp-sass": "^4.1.0",
    "gulp-swig": "^0.9.1",
    "gulp-uglify": "^3.0.2",
    "gulp-useref": "^5.0.0"
  }
```

到这里我们的起始工作就算完成了，接下来就需要调试使用这个模块去解决任务中要处理的一些问题，比如路径匹配错误啥的；

接下来删除原项目 `gulpfile.js` 的所有内容和 `devDependencies` 所有依赖，删除原项目中依赖文件夹（`node_modules`）和构建任务生成的文件夹（`dist` 和 `temp`）；

在模块根目录下将模块link到全局；

![](03.jpg)

在项目根目录下，将模块link到当前项目；此时，项目下会多出 `node_modules` ，并添加了模块软连接；

![](04.jpg)

![](05.jpg)

此时到项目 `gulpfile.js` 下将模块导入并作为导出，即此时 `gulpfile.js` 又导出了之前上一篇文章中我们创建的构建任务；

```js
module.exports = require('cs-pages')
```

# 解决模块中的问题

到这一步我们就可以开始执行去调试修改相关任务了，这里我们先执行 **yarn build**；

此时因为项目依赖没有 `gulp`，会报找不到 `gulp` 的错误，先暂时引入依赖解决这个问题；

再次执行 **yarn build**，继续报错，我们继续定位解决；

![](06.jpg)

查看代码，此处是我们 `swig` 要解析的模板数据 `data` ，因为此时在模块中，所以找不到相应的 `package.json` ，而这个 `data` 是项目所需要的，不应该放在模板下；

解决办法：在项目目录项创建配置文件 `pages.config.js` ，把 `data` 拷贝进来并导出，后续在模块中去读取这个配置文件的内容；

![](07.jpg)

`index.js` 修改内容，删除 `data` ，并读取 `pages.config.js`

```js
const cwd = process.cwd()   // 返回命令行的工作目录，此处即在项目中运行时的项目根目录
let config = {
    // default
}

try {
    const loadConfig = require(`${cwd}/pages.config.js`)
    config = Object.assign({}, config, loadConfig)
} catch (e) {

}
```

再次 **yarn build**，继续报错，继续解决：

![](08.jpg)

将 `babel` 的 `presets` 配置 `'@babel/preset-env'` 改为 `require` 导入；

```js
// 脚本编译任务
const script = () => {
    return src('src/assets/scripts/*.js', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.babel({presets: [require('@babel/preset-env')]}))  // babel转换   记住presets参数要传
        .pipe(dest('temp'))
        .pipe(bs.reload({stream: true}))
}
```

再次 **yarn build**，此时构建OK！！！赞！查看相应文件，也做了压缩处理。

![](09.jpg)

# 抽象路径配置

接下来我们再对代码中写死的一些路径进行更深度的封装，让其支持可配置，使代码路径更灵活；在 `config` 中配置默认路径参数，项目的 `pages.config.js` 中也可添加对应参数，并可根据实际项目的路径修改对应的值，去执行构建任务；

此时模块 `index.js` 为：（注意 `config` 中的默认build配置）

```js
const {src, dest, parallel, series, watch} = require('gulp')
const del = require('del')
const loadPlugins = require('gulp-load-plugins')    // 导入gulp-load-plugins，它是一个方法
const plugins = loadPlugins()   // 获取plugins对象，属性是各个gulp插件名除去gulp-开头后的部分，后续有短横线使用驼峰

const browserSync = require('browser-sync')
const bs = browserSync.create()
const cwd = process.cwd()   // 返回命令行的工作目录，此处即在项目中运行时的项目根目录
let config = {
    // default config
    build: {
        src: 'src',
        dist: 'dist',
        temp: 'temp',
        public: 'public',
        paths: {
            styles: 'assets/styles/*.scss',
            scripts: 'assets/scripts/*.js',
            pages: '*.html',
            images: 'assets/images/**',
            fonts: 'assets/fonts/**'
        }
    }
}

try {
    const loadConfig = require(`${cwd}/pages.config.js`)
    config = Object.assign({}, config, loadConfig)
} catch (e) {

}

// 样式编译任务
const style = () => {
    return src(config.build.paths.styles, {base: config.build.src, cwd: config.build.src}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.sass({outputStyle: 'expanded'}))       // sass转换，参数： {outputStyle: 'expanded'}  完全展开
        .pipe(dest(config.build.temp))
        .pipe(bs.reload({stream: true}))
}

// 脚本编译任务
const script = () => {
    return src(config.build.paths.scripts, {base: config.build.src, cwd: config.build.src}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.babel({presets: [require('@babel/preset-env')]}))  // babel转换   记住presets参数要传
        .pipe(dest(config.build.temp))
        .pipe(bs.reload({stream: true}))
}

// 页面编译任务
const page = () => {
    return src(config.build.paths.pages, {base: config.build.src, cwd: config.build.src})     // 'src/**/*.html'可创建到对应路径，此处都放在src下边
        .pipe(plugins.swig({data: config.data, defaults: {cache: false}}))     // cache:false防止模板缓存导致页面不能及时更新
        .pipe(dest(config.build.temp))
        .pipe(bs.reload({stream: true}))
}

// 图片转换
const image = () => {
    return src(config.build.paths.images, {base: config.build.src, cwd: config.build.src}, {base: 'src'})
        .pipe(plugins.imagemin())
        .pipe(dest(config.build.temp))
}

// 字体文件转换
const font = () => {
    return src(config.build.paths.fonts, {base: config.build.src, cwd: config.build.src})
        .pipe(plugins.imagemin())
        .pipe(dest(config.build.dist))
}

// public下
const extra = () => {
    return src('**', {base: config.build.public, cwd: config.build.public})
        .pipe(dest(config.build.dist))
}

// 清除dist
const clean = () => {
    return del([config.build.dist, config.build.temp])    // del参数是路径集合
}

// 热更新服务
const serve = () => {
    // 监听以下文件路径的通配符，文件修改后执行对应任务
    watch(config.build.paths.styles, {cwd: config.build.src}, style)
    watch(config.build.paths.scripts, {cwd: config.build.src}, script)
    watch(config.build.paths.pages, {cwd: config.build.src}, page)
    // watch('src/assets/images/**', image)     // 对于这些静态资源开发阶段的监听意义不大，反而增加了任务开销，在bs中添加baseDir初始化就行
    // watch('src/assets/fonts/**', font)
    // watch('public/**', extra)
    watch([                         // 对于静态资源改变也需要热更新，不需要编译转换，只需要服务刷新就行
        config.build.paths.images,
        config.build.paths.fonts,
    ], {cwd: config.build.src}, bs.reload)
    watch('**', {cwd: config.build.public}, bs.reload)    // public下的静态资源

    bs.init({
        notify: false,  // 服务启动时，浏览器右上角的提示关闭
        port: 2080,     // 默认值 3000
        // open: false, // 启动时是否自动打开浏览器，默认值true
        // files: 'dist/**',      // 服务器启动后监听的路径通配符，对应的文件修改后就可以热更新;在对应任务后使用bs.reload就可以不使用files监听
        server: {
            baseDir: [config.build.temp, config.build.src, config.build.public],     // 服务代码的根目录
            routes: {
                '/node_modules': 'node_modules'  // 路径映射，优于baseDir，相关配置找不到再找baseDir
            }
        }
    })
}

const useref = () => {
    return src(`${config.build.temp}/${config.build.paths.pages}`, {base: config.build.temp})   // 此处暂时未用cwd标识根路径，原因是使用后无法生成vendor.css等文件
        .pipe(plugins.useref({searchPath: [config.build.temp, '.']}))
        .pipe(plugins.if(/\.html$/, plugins.htmlmin({
            collapseWhitespace: true,
            minifyCSS: true,
            minifyJS: true
        })))     // html压缩
        .pipe(plugins.if(/\.css$/, plugins.cleanCss()))     // css压缩
        .pipe(plugins.if(/\.js$/, plugins.uglify()))        // js压缩
        .pipe(dest(config.build.dist))          // 目标文件夹由temp改为dist，防止读写冲突导致读写失败
}

// 组合任务
const compile = parallel(style, script, page)      // 编译
const build = series(            // 构建
    clean,
    parallel(
        series(compile, useref),
        image,
        font,
        extra
    )
)
const develop = series(compile, serve)      // 开发阶段构建+热更新服务

module.exports = {
    clean,
    build,
    develop,
}
```

`pages.config.js` 中添加对应 `build` 路径配置，可根据实际需要修改配置，如修改打包目标路径为 `release` 目录，临时目录为 `dist` 目录，执行构建任务后得到 `release` 目录，点击 `index.html` ，打开项目成功；

![](10.jpg)

# 包装 Gulp Cli

以上我们基于 `gulp` 的自动化构建工作流基本完成，但是为了能更加便捷，我们可以把每次要添加 `gulpfile` 这个操作也包装进模块当中，思路就是 `gulp` 提供了相关的命令行参数让我们去指定 `gulpfile` 的路径；

我们先删除相应的编译文件夹（`release` 和 `dist`）以及 `gulpfile.js` 后来测试一下；执行 **yarn gulp build --gulpfile ./node_modules/cs-pages/lib/index.js --cwd .**

我们来解读一下这个命令，**yarn gulp build** 执行编译构建任务，**--gulpfile ./node_modules/cs-pages/lib/index.js** 指定 `gulpfile` 所在路径，此时正是我们模块的软链接，**--cwd .** 指定目录为当前运行目录，否则 `gulp` 会默认是 `gulpfile` 所在目录，即软连接的lib目录；

执行后同样能编译得到 `release` 和 `dist` 目录；

那么问题来了，如果不用 `gulpfile` ，我们每次都需要指定路径去构建，那这个命令也太长了点，接下来便有了将命令封装到 `cli `中的方法，将 `gulp` 都封装到模块中；

首先到模块下创建 `cli` 程序；

![](11.jpg)

![](12.jpg)

测试 `cli` 是否成功；

进入模块下，重新 `link` 一下模块，然后执行 `cli` 命令，测试打印结果；

![](13.jpg)

打印OK，接下来我们就可以去修改 `cli` 对 `gulpfile` 的读取进行封装；

```js
#!/usr/bin/env node

// 指定cwd工作目录和gulpfile路径
process.argv.push('--cwd')
process.argv.push(process.cwd())
process.argv.push('--gulpfile')
process.argv.push(require.resolve('..'))    // require.resolve当前模块的路径，gulpfile就是../lib/index.js, 改为..就会自动到package.json中找main字段的值

require('gulp/bin/gulp')        // 我们的cli中导入gulp-cli的执行，即导入gulp-cli并执行
```

接下来我们到项目下删除 `release` 和 `dist` ，并使用 `cli` 执行构建任务 `cs-pages build` ，任务OK，同样编译生成 `release` 和`dist` ！！！

![](14.jpg)

# 发布并使用模块

在 `package.json` 中添加配置项 `files`，将 `bin` 和 `lib` 目录加进去，因为在发布时，会发布 `files` 下的目录以及根目录下的其他内容；

将代码提交到 `git` 仓库，然后执行 **yarn publish --registry [https://registry.yarnpkg.com](https://link.zhihu.com/?target=https%3A//registry.yarnpkg.com)**；成功后此时 `npm` 上已经有属于你自己的 `cli` 包了，赞！

![](15.jpg)

![](16.jpg)

接下来我们去测试使用一下自己的 `cli` 吧；

创建一个空的文件夹（我这使用 `page-gulp-cli-demo` ），去之前项目中把项目结构（ `public`，`src`，`pages.config.js`）拷贝进来，然后初始化一下 `package.json` 文件，

将自己的 `cli` 包安装到开发依赖 **yarn add cs-pages --dev**，本人第一次发布使用自己的包，成就感满满=。=！

![](17.jpg)

执行 **cs-pages build** 等构件任务均可完成！

最后我还可以对当前项目使用这个 `cli` 的命令进行简化，将其写到 `package.json` 中去；

![](18.jpg)

此时只需要执行 **yarn build** 等命令即可，大赞！

以上，结合之前的案例，整个项目的自动化构建任务工作流完整结束，后续如果使用gulp做项目打包工具，根据项目需要去编写自己的项目的工作流即可。

相关链接：

[前端工程化之Gulp](https://seachan0117.github.io/2021/06/11/gulp-01/)

[Gulp案例](https://seachan0117.github.io/2021/06/11/gulp-02/)

整个过程中使用的基础项目来自 zce 大牛：

[zce/zce-gulp-demo](https://link.zhihu.com/?target=https%3A//github.com/zce/zce-gulp-demo)
