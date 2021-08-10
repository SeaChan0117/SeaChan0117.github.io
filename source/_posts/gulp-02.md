---
title: Gulp案例
date: 2021-08-10 09:23:06
categories:
- 大前端
- 工程化
tags:
- gulp
---
![gulp](bg.jpg)

[上一篇](https://seachan0117.github.io/2021/06/11/gulp-01)，我们学习了 `Gulp` 的一些基本原理和操作，本文通过小案例，去学习构建一个网页应用的自动化构建工作流。

首先准备一个用于构建网页应用的项目，项目目录大致如下，大家可以使用自己的；

![项目目录](项目目录.jpg)

接下来安装gulp到开发依赖，在根目录下新建 `gulpfile.js` 文件；

# 样式编译任务

样式文件读取拷贝到目标文件夹，`src` 第二个参数，`base` 之后的路径会按照原路径保存在 `dist` 下，否则所有文件都会直接放在 `dist`下；

```js
const {src, dest} = require('gulp')

const style = () => {
    return src('src/assets/styles/*.scss', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(dest('dist'))
}

module.exports = {  // 所有的任务都放这
    style
}
```

以上执行任务后得到的文件是原文件的拷贝（*.scss），还未完成转换，此时需要安装插件对文件流进行转换，`yarn add gulp-sass --dev`;

```js
const {src, dest} = require('gulp')
const sass = require('gulp-sass')
const style = () => {
    return src('src/assets/styles/*.scss', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(sass({outputStyle: 'expanded'}))       // sass转换，参数： {outputStyle: 'expanded'}  完全展开
        .pipe(dest('dist'))
}

module.exports = {  // 所有的任务都放这
    style,
}
```

执行 `yarn gulp style` 编译完成样式文件，可在 `dist` 下查看；

PS: 下划线开头的文件，`sass` 插件会忽略，不会进行转换；

![样式编译任务](样式编译任务.jpg)

# 脚本文件编译任务

安装转换流插件，`yarn add gulp-babel --dev`；执行脚本任务时，会报错 `Error: Cannot find module '@babel/core'`，是因为 `gulp-babel` 模块并没有转换功能，只是帮我们唤起 `@babel/core` 转换模块，我们需要手动安装，`yarn add @babel/core @babel/preset-env --dev`；

```js
const {src, dest} = require('gulp')
const babel = require('gulp-babel')

// 脚本编译任务
const script = () => {
    return src('src/assets/scripts/*.js', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(babel({presets: ['@babel/preset-env']}))  // babel转换   记住presets参数要传
        .pipe(dest('dist'))
}
module.exports = {  // 所有的任务都放这
    style,
    script,
}
```

执行 `yarn gulp script` 编译完成脚本文件，可在 `dist` 下查看；

# 页面文件编译任务

为了让页面中一些可重用的部分抽象出来，可以使用模板引擎，案例中使用 `swig`，需要先安装 `swig` 的转换插件，`yarn add gulp-swig --dev`；

![页面编译任务](页面编译任务.jpg)

```js
const {src, dest} = require('gulp')
const swig = require('gulp-swig')

// 模拟模板数据，swig配置解析，推荐使用json文件，再读取载入进来
const data = {
    menus: [
        {
            name: 'Home',
            icon: 'aperture',
            link: 'index.html'
        },
        {
            name: 'Features',
            link: 'features.html'
        },
        {
            name: 'About',
            link: 'about.html'
        },
        {
            name: 'Contact',
            link: '#',
            children: [
                {
                    name: 'Twitter',
                    link: 'https://twitter.com'
                },
                {
                    name: 'About',
                    link: 'https://weibo.com'
                },
                {
                    name: 'divider'
                },
                {
                    name: 'GitHub',
                    link: 'https://github.com'
                }
            ]
        }
    ],
    pkg: require('./package.json'),
    date: new Date()
}
// 页面编译任务
const page = () => {
    return src('src/*.html', {base: 'src'})     // 'src/**/*.html'可创建到对应路径，此处html都放在src下边，避免干扰，其他文件夹下的是布局页，不影响
        .pipe(swig({data}))
        .pipe(dest('dist'))
}
module.exports = {  // 所有的任务都放这
    style,
    script,
    page,
}
```

执行 `yarn gulp page` 编译完成页面文件，可在 `dist` 下查看；

以上完成了样式 + 脚本 + 页面文件的单个构建任务，接下来创建组合任务让他们都一次执行，因为三个任务互不干扰，可以创建并行任务，提高效率；

```js
// 组合任务
const {parallel} = require('gulp')
const compile = parallel(style, script, page)
module.exports = {
    compile
}
```

执行 `yarn gulp compile` 得到组合任务结果：

![组合任务](组合任务.jpg)

# 图片和字体文件的转换

图片文件压缩转换，需要添加插件，`yarn add gulp-imagemin --dev`;

```js
const imagemin = require('gulp-imagemin')
// 图片转换
const image = () => {
    return src('src/assets/images/**', {base: 'src'})
        .pipe(imagemin())
        .pipe(dest('dist'))
}

// 字体文件转换
const font = () => {
    return src('src/assets/fonts/**', {base: 'src'})
        .pipe(imagemin())
        .pipe(dest('dist'))
}

// 组合任务
const compile = parallel(style, script, page, image, font)
module.exports = {
    compile,
}
```

# 其他文件及文件清除

`public` 目录下的文件；

```js
// public下
const extra = () => {
    return src('public/**', {base: 'public'})
        .pipe(dest('dist'))
}

// 组合任务
const compile = parallel(style, script, page, image, font)      // 编译
const build = parallel(compile, extra)      // 构建
module.exports = {
    compile,
    build,
}
```

目前整个项目的构建任务基本完成了，但每次执行构建任务都要手动删除 `dist` 目录，我们使用一个插件来完成每次的自动删除；安装 `yarn add del --dev`；定义添加一个 `clean` 任务，用于每次执行构建任务前操作，此时的构建就需要 `series` 创建一个串行任务，先清空再执行后边的构建任务，然后我们的任务变成了这样：

```js
// 清除dist
const del  = require('del')
const clean = () => {
    return del(['dist'])    // del参数是路径集合
}
// 组合任务
const compile = parallel(style, script, page, image, font)      // 编译
const build = series(clean, parallel(compile, extra))    // 构建
module.exports = {
    compile,
    build,
}
```

# 自动加载插件

随着构建越来越复杂，其中使用到的插件就越来越多，当我们 `require` 一堆插件时，代码开始显得很繁杂，我们可以通过一个插件去管理 `yarn add gulp-load-plugins --dev`；导入插件后对之前使用到的插件做相应替换，代码变成了下边的样子：

```js
const {src, dest, parallel, series} = require('gulp')
const loadPlugins = require('gulp-load-plugins')    // 导入gulp-load-plugins，它是一个方法
const plugins = loadPlugins()   // 获取plugins对象，属性是各个gulp插件名除去gulp-开头后的部分，后续有短横线使用驼峰
const del  = require('del')

// 模板数据，swig配置解析，推荐使用json文件，再读取载入进来
const data = {
    menus: [
        {
            name: 'Home',
            icon: 'aperture',
            link: 'index.html'
        },
        {
            name: 'Features',
            link: 'features.html'
        },
        {
            name: 'About',
            link: 'about.html'
        },
        {
            name: 'Contact',
            link: '#',
            children: [
                {
                    name: 'Twitter',
                    link: 'https://twitter.com'
                },
                {
                    name: 'About',
                    link: 'https://weibo.com'
                },
                {
                    name: 'divider'
                },
                {
                    name: 'GitHub',
                    link: 'https://github.com'
                }
            ]
        }
    ],
    pkg: require('./package.json'),
    date: new Date()
}

// 样式编译任务
const style = () => {
    return src('src/assets/styles/*.scss', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.sass({outputStyle: 'expanded'}))       // sass转换，参数： {outputStyle: 'expanded'}  完全展开
        .pipe(dest('dist'))
}

// 脚本编译任务
const script = () => {
    return src('src/assets/scripts/*.js', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.babel({presets: ['@babel/preset-env']}))  // babel转换   记住presets参数要传
        .pipe(dest('dist'))
}

// 页面编译任务
const page = () => {
    return src('src/*.html', {base: 'src'})     // 'src/**/*.html'可创建到对应路径，此处都放在src下边
        .pipe(plugins.swig({data}))
        .pipe(dest('dist'))
}

// 图片转换
const image = () => {
    return src('src/assets/images/**', {base: 'src'})
        .pipe(plugins.imagemin())
        .pipe(dest('dist'))
}

// 字体文件转换
const font = () => {
    return src('src/assets/fonts/**', {base: 'src'})
        .pipe(plugins.imagemin())
        .pipe(dest('dist'))
}

// public下
const extra = () => {
    return src('public/**', {base: 'public'})
        .pipe(dest('dist'))
}

// 清除dist
const clean = () => {
    return del(['dist'])    // del参数是路径集合
}

// 组合任务
const compile = parallel(style, script, page, image, font)      // 编译
const build = series(clean, parallel(compile, extra))    // 构建

module.exports = {
    compile,
    build,
}
```

# 热更新开发服务器

以上是代码的构建基本流程，除了代码构建，我们还需要在开发阶段启动一个服务器用于调试代码；`gulp` 也为我们提供了相应的功能，配合其他的构建任务，在我们代码修改过后完成自动的编译，自动地刷新浏览器页面；

首先安装一个 `browser-sync` 的模块，`yarn add browser-sync --dev`；

```js
const browserSync = require('browser-sync')
const bs = browserSync.create()
// 热更新服务
const serve = () => {
    bs.init({
        notify: false,  // 服务启动时，浏览器右上角的提示关闭
        port: 2080,     // 默认值 3000
        // open: false, // 启动时是否自动打开浏览器，默认值true
        files: 'dist/**',      // 服务器启动后监听的路径通配符，对应的文件修改后就可以热更新
        server: {
            baseDir: 'dist',     // 服务代码的根目录
            routes: {
                '/node_modules': 'node_modules'  // 路径映射，优于baseDir，相关配置找不到再找baseDir
            }
        }
    })
}
module.exports = {
    compile,
    build,
    serve,
}
```

`yarn gulp serve` 执行任务后，浏览器自动启动并打开项目默认页面，修改 `dist` 目录下的文件，页面能实时刷新；

# 监视变化以及构建优化

现在我们有了开发服务器，也能实时监视 `dist` 目录下的变化实时刷新浏览器，但实际开发过程中，我们修改的代码并不是 `dist` 目录，此时我们还需要另一个插件 `watch` 来帮助监听某个路径下的变化来决定是否要重新执行的任务；

```js
const {src, dest, parallel, series, watch} = require('gulp')  // 导入watch
// 热更新服务
const serve = () => {
    // 监听以下文件路径的通配符，文件修改后执行对应任务
    watch('src/assets/styles/*.scss', style)
    watch('src/assets/scripts/*.js', script)
    watch('src/*.html', page)
    watch('src/assets/images/**', image)
    watch('src/assets/fonts/**', font)
    watch('public/**', extra)

    bs.init({
        notify: false,  // 服务启动时，浏览器右上角的提示关闭
        port: 2080,     // 默认值 3000
        // open: false, // 启动时是否自动打开浏览器，默认值true
        files: 'dist/**',      // 服务器启动后监听的路径通配符，对应的文件修改后就可以热更新
        server: {
            baseDir: 'dist',     // 服务代码的根目录
            routes: {
                '/node_modules': 'node_modules'  // 路径映射，优于baseDir，相关配置找不到再找baseDir
            }
        }
    })
}
```

保存后重启 `serve` 任务，修改 `src` 下的任意文件，浏览器即可实时同步刷新。

以上完成后还需对任务做一些调整，将静态资源（图片字体等）的构建任务放在 `build` 下，然后将 `compile` 和 `serve` 放一起成为开发阶段的组合任务 `develop`；

```js
// 热更新服务
const serve = () => {
    // 监听以下文件路径的通配符，文件修改后执行对应任务
    watch('src/assets/styles/*.scss', style)
    watch('src/assets/scripts/*.js', script)
    watch('src/*.html', page)
    // watch('src/assets/images/**', image)     // 对于这些静态资源开发阶段的监听意义不大，反而增加了任务开销，在bs中添加baseDir初始化就行
    // watch('src/assets/fonts/**', font)
    // watch('public/**', extra)
    watch([                         // 对于静态资源改变也需要热更新，不需要编译转换，只需要服务刷新就行
        'src/assets/images/**',
        'src/assets/fonts/**',
        'public/**',
    ], bs.reload)

    bs.init({
        notify: false,  // 服务启动时，浏览器右上角的提示关闭
        port: 2080,     // 默认值 3000
        // open: false, // 启动时是否自动打开浏览器，默认值true
        files: 'dist/**',      // 服务器启动后监听的路径通配符，对应的文件修改后就可以热更新
        server: {
            baseDir: ['dist', 'src', 'public'],     // 服务代码的根目录
            routes: {
                '/node_modules': 'node_modules'  // 路径映射，优于baseDir，相关配置找不到再找baseDir
            }
        }
    })
}

// 组合任务
const compile = parallel(style, script, page)      // 编译
const build = series(clean, parallel(compile, image, font, extra))    // 构建
const develop = series(compile, serve)      // 开发阶段构建+热更新服务

module.exports = {
    compile,
    build,
    develop,
}
```

此时执行启动 `develop` 任务后，生成的 `dist` 下图片字体等静态资源的文件是没有的，通过 `baseDir` 下从 `src` 和 `public` 下取，开发过程中减少了构建任务；

`bs.init` 中监听files的参数可以去掉，改为对应的任务后使用 `.pipe(bs.reload({stream:true}))` ，结果一样可以得到想要的热更新，只是写法上的差异；

```js
// 样式编译任务
const style = () => {
    return src('src/assets/styles/*.scss', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.sass({outputStyle: 'expanded'}))       // sass转换，参数： {outputStyle: 'expanded'}  完全展开
        .pipe(dest('dist'))
        .pipe(bs.reload({stream: true}))
}

// 脚本编译任务
const script = () => {
    return src('src/assets/scripts/*.js', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.babel({presets: ['@babel/preset-env']}))  // babel转换   记住presets参数要传
        .pipe(dest('dist'))
        .pipe(bs.reload({stream: true}))
}

// 页面编译任务
const page = () => {
    return src('src/*.html', {base: 'src'})     // 'src/**/*.html'可创建到对应路径，此处都放在src下边
        .pipe(plugins.swig({data, defaults: { cache: false }}))     // cache:false防止模板缓存导致页面不能及时更新
        .pipe(dest('dist'))
        .pipe(bs.reload({stream: true}))
}
```

# useref 引用文件处理

通过 `build` 任务我们需要得到一个最终发布版本的包，但之前我们在 `bs` 服务中做了一个路由映射处理了 `node_modules` 下边的依赖，如果线上使用该构建包，相应的依赖是找不到的，现在需要对它们进一步处理；

我们使用一个插件 `useref` ，`yarn add gulp-useref -- dev` ，它能根据对应的注释模板，将其中相应引入路径下的文件，编译转换后放到注释中规定的目录文件中；如下，会将 ` "/node_modules/bootstrap/dist/css/bootstrap.css"`  找到并编译压缩后放在 `assets/styles/vendor.css` 中，如果 `build` 和 `endbuild` 之间有多个文件，将统一放在一个文件中；

```html
<!-- build:css assets/styles/vendor.css -->
<link rel="stylesheet" href="/node_modules/bootstrap/dist/css/bootstrap.css">
<!-- endbuild -->
```

添加 `useref` 任务

```js
const useref = () => {
    return src('dist/*.html', {base: 'dist'})
        .pipe(plugins.useref({searchPath: ['dist', '.']}))
        .pipe(dest('dist'))
}
module.exports = {
    compile,
    build,
    develop,
    useref,
}
```

执行 `useref` 任务后，`dist/assets/styles` 下的多了 `vendor.css` ，`dist/assets/scripts` 下多了 `vendor.js` ，`html` 中涉及`node_modules` 的依赖也已相应转换；

![useref任务](useref任务01.jpg)

![useref任务](useref任务02.jpg)

![useref任务](useref任务03.jpg)

# 文件压缩

上边使用 `useref` 后，我们已经能得到相关的 `css` 和 `js` 文件，但是都是内容全拷贝过来的，我们还可以对其进行一系列的压缩减少构建包的大小；针对 `html`，`css`，`js` 安装相应的插件进行压缩，`yarn add gulp-htmlmin gulp-clean-css gulp-uglify --dev` ，此时还需要一个判断文件类型的插件，`yarn add gulp-if --dev` ，判断文件后针对不同文件使用不同的压缩插件；

```js
const useref = () => {
    return src('dist/*.html', {base: 'dist'})
        .pipe(plugins.useref({searchPath: ['dist', '.']}))
        .pipe(plugins.if(/\.html$/, plugins.htmlmin({
            collapseWhitespace: true,
            minifyCSS: true,
            minifyJS: true
        })))     // html压缩
        .pipe(plugins.if(/\.css$/, plugins.cleanCss()))     // css压缩
        .pipe(plugins.if(/\.js$/, plugins.uglify()))        // js压缩
        .pipe(dest('release'))          // 目标文件夹由dist改为其他，防止读写冲突导致读写失败
}
```

重新执行 `compile` 和 `useref` 到 `release` 目录查看相应文件吧；

## 重新规划构建过程

使用 `useref` 后，我们之前的构建结构被打破了，我们之前定好的打包目录是 `dist` ，但是使用 `useref` 时为了解决文件冲突，我们重新放在了 `release` 目录，即 `release` 成了要上线的包文件，然而 `release` 下现在又没有图片字体等静态资源文件，所以需要对构建任务进行重新规划；思路：之前的任务使用 `temp` 临时目录，最后将 `temp` 目录下的文件转化后归整到 `dist` ；

调整后代码为：

```js
const {src, dest, parallel, series, watch} = require('gulp')
const del = require('del')
const loadPlugins = require('gulp-load-plugins')    // 导入gulp-load-plugins，它是一个方法
const plugins = loadPlugins()   // 获取plugins对象，属性是各个gulp插件名除去gulp-开头后的部分，后续有短横线使用驼峰

const browserSync = require('browser-sync')
const bs = browserSync.create()

// 模板数据，swig配置解析，推荐使用json文件，再读取载入进来
const data = {
    menus: [
        {
            name: 'Home',
            icon: 'aperture',
            link: 'index.html'
        },
        {
            name: 'Features',
            link: 'features.html'
        },
        {
            name: 'About',
            link: 'about.html'
        },
        {
            name: 'Contact',
            link: '#',
            children: [
                {
                    name: 'Twitter',
                    link: 'https://twitter.com'
                },
                {
                    name: 'About',
                    link: 'https://weibo.com'
                },
                {
                    name: 'divider'
                },
                {
                    name: 'GitHub',
                    link: 'https://github.com'
                }
            ]
        }
    ],
    pkg: require('./package.json'),
    date: new Date()
}

// 样式编译任务
const style = () => {
    return src('src/assets/styles/*.scss', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.sass({outputStyle: 'expanded'}))       // sass转换，参数： {outputStyle: 'expanded'}  完全展开
        .pipe(dest('temp'))
        .pipe(bs.reload({stream: true}))
}

// 脚本编译任务
const script = () => {
    return src('src/assets/scripts/*.js', {base: 'src'}) // base之后的路径会按照原路径(assets/styles/*)保存在dist下，否则所有文件都在dist下
        .pipe(plugins.babel({presets: ['@babel/preset-env']}))  // babel转换   记住presets参数要传
        .pipe(dest('temp'))
        .pipe(bs.reload({stream: true}))
}

// 页面编译任务
const page = () => {
    return src('src/*.html', {base: 'src'})     // 'src/**/*.html'可创建到对应路径，此处都放在src下边
        .pipe(plugins.swig({data, defaults: {cache: false}}))     // cache:false防止模板缓存导致页面不能及时更新
        .pipe(dest('temp'))
        .pipe(bs.reload({stream: true}))
}

// 图片转换
const image = () => {
    return src('src/assets/images/**', {base: 'src'})
        .pipe(plugins.imagemin())
        .pipe(dest('dist'))
}

// 字体文件转换
const font = () => {
    return src('src/assets/fonts/**', {base: 'src'})
        .pipe(plugins.imagemin())
        .pipe(dest('dist'))
}

// public下
const extra = () => {
    return src('public/**', {base: 'public'})
        .pipe(dest('dist'))
}

// 清除dist
const clean = () => {
    return del(['dist', 'temp'])    // del参数是路径集合
}

// 热更新服务
const serve = () => {
    // 监听以下文件路径的通配符，文件修改后执行对应任务
    watch('src/assets/styles/*.scss', style)
    watch('src/assets/scripts/*.js', script)
    watch('src/*.html', page)
    // watch('src/assets/images/**', image)     // 对于这些静态资源开发阶段的监听意义不大，反而增加了任务开销，在bs中添加baseDir初始化就行
    // watch('src/assets/fonts/**', font)
    // watch('public/**', extra)
    watch([                         // 对于静态资源改变也需要热更新，不需要编译转换，只需要服务刷新就行
        'src/assets/images/**',
        'src/assets/fonts/**',
        'public/**',
    ], bs.reload)

    bs.init({
        notify: false,  // 服务启动时，浏览器右上角的提示关闭
        port: 2080,     // 默认值 3000
        // open: false, // 启动时是否自动打开浏览器，默认值true
        // files: 'dist/**',      // 服务器启动后监听的路径通配符，对应的文件修改后就可以热更新;在对应任务后使用bs.reload就可以不使用files监听
        server: {
            baseDir: ['temp', 'src', 'public'],     // 服务代码的根目录
            routes: {
                '/node_modules': 'node_modules'  // 路径映射，优于baseDir，相关配置找不到再找baseDir
            }
        }
    })
}

const useref = () => {
    return src('temp/*.html', {base: 'temp'})
        .pipe(plugins.useref({searchPath: ['temp', '.']}))
        .pipe(plugins.if(/\.html$/, plugins.htmlmin({
            collapseWhitespace: true,
            minifyCSS: true,
            minifyJS: true
        })))     // html压缩
        .pipe(plugins.if(/\.css$/, plugins.cleanCss()))     // css压缩
        .pipe(plugins.if(/\.js$/, plugins.uglify()))        // js压缩
        .pipe(dest('dist'))          // 目标文件夹由dist改为其他，防止读写冲突导致读写失败
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

# 构建任务补充

将导出的任务定义到 `package.json` 的 `scripts` 中，使用 `yarn clean` 等命令执行任务；

![scripts命令](scripts命令.jpg)

在 `.ignore` 文件中添加 `temp` 和 `dist` 目录；

以上就是使用 `gulp` 创建构建任务的整个流程，只要把这个作为模板使用就行，后续我们还可以对工作流进行封装发布，我们下一篇再见。console.log('Gulp ♥♥♥')

