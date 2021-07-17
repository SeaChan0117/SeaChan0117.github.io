---
title: Vue3 介绍
date: 2021-07-17 15:59:08
categories:
tags:
---

# Vue.js 3.0 源码组织方式
1.源码采用 TypeScript 重写，语言类型化提升代码的可维护性；

2.使用 Monorepo 管理项目结构，把不同功能的代码放在不同的 package 中管理，功能模块划分明确，依赖关系明确，功能模块也可单独使用；

# Vue.js 3.0 不同构建版本

## cjs

完整版的 vue ，包含运行时和编译器；

### vue.cjs.js

开发版本，代码未没有被压缩；

### vue.cjs.prod.js

生产版本，代码被压缩；

## global

以下都可以在浏览器中使用 script 标签进行导入，导入后会有一个全局的 Vue 对象；

### vue.global.js

完整版的 vue，包含编译器和运行时，开发版本，代码没有压缩；

### vue.global.prod.js

完整版的 vue，包含编译器和运行时，生产版本，代码被压缩；

### vue.runtime.global.js

只包含运行时的构建版本，开发版本，代码没有压缩；

### vue.runtime.global.prod.js

只包含运行时的构建版本，生产版本，代码被压缩；

## browser

ESModule，浏览器原生模块化方式，在浏览器中使用 script 标签，标签设置 type 为 module 导入以下模块；

### vue.esm-browser.js

完整版的 vue，包含编译器和运行时，开发版本，代码没有压缩；

### vue.esm-browser.prod.js

完整版的 vue，包含编译器和运行时，生产版本，代码被压缩；

### vue.runtime.esm-browser.js

只包含运行时的构建版本，开发版本，代码没有压缩；

### vue.runtime.esm-browser.prod.js

只包含运行时的构建版本，生产版本，代码被压缩；

## bundler

没有打包所有的代码，需要配合打包工具使用；

### vue.esm-bundler.js

包含运行时编译器；

vue.runtime.esm-bundler.js

 仅运行时，并要求所有模板都要预先编译；

# Composition API

* FRC（Request For Comments）
  * [https://github.com/vuejs/rfcs](https://github.com/vuejs/rfcs)
* Composition API RFC
  * [https://composition-api.vuejs.org](https://composition-api.vuejs.org)

## options API

1.包含一个描述组件选项（data、methods、props 等）的对象；

2.Options API 开发复杂组件，同一个功能逻辑的代码被拆分到不同选项；

composition API

1.基于函数的 API ，更灵活地组织组件的逻辑；

2.更利于公共逻辑的提取和重用；

# 性能提升

## 响应式系统升级

* Vue.js 2.x 中响应式系统的核心是 defineProperty；
* Vue.js 3.0 中使用 Proxy 对象重写响应式系统，性能本身比 defineProperty 好；
  * 可以监听动态新增的属性；
  * 可以监听删除的属性；
  * 可以监听数组的索引和 length 属性；

## 编译优化

* Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程（静态节点还会 diff）；
* Vue.js 3.0 中标记和提升所有的静态根节点，diff 的时候只需要对比动态节点内容；
  * Fragments (升级 vetur 插件)；
  * 静态提升；
  * Patch flag；
  * 缓存事件处理函数；

## 优化打包体积

* Vue.js 3.0 中移除了一些不常用的 API；
  * eg：inline-template、filter 等
* Tree-shaking；

# Vite

## vite 和 vue-cli 区别

* Vite 在开发模式下不需要打包可以直接运行（ESModule 的加载特性）；
* Vue-cli 开发模式下必须对项目打包才可以运行；
* Vite 在生产环境下使用 Rollup 打包；
  * 基于 ES Module 的方式打包；
* Vue-cli 使用 webpack 打包；

## Vite 特点

* 快速冷启动；
* 按需编译；
* 模块热更新；

## Vite 创建项目

* 一般方式：

  ​	```npm init vite-app <project-name>```

* 基于模板创建项目：

  ​	```npm init vite-app --template react```

  ​	```npm init vite-app --template preact```

