---
title: Plop 使用
date: 2021-11-03 21:10:33
categories:
- 大前端
- 工程化
tags:
- plop

cover: https://tse2-mm.cn.bing.net/th/id/OIP-C.D1UgmRsLJnX-BuaETMpLEwHaFO?pid=ImgDet&rs=1
---

# 什么是 Plop
创建项目中特定文件类型的小工具，类似于Yeoman中的sub generator，一般不会独立使用。一般会把Plop集成到项目中，用来自动化的创建同类型的项目文件。

[Plop](https://plopjs.com/documentation/)


# 使用
以 Vue 为例，通过 Plop 生成对应页面；

## 安装
```sh
yarn add plop -D
```

## 配置
* 项目根目录下创建 `plopfile.js` 的入口文件；
> Plop 入口文件，需要导出一个函数，此函数接收一个 plop 对象，用于创建生成器任务

```js
// Plop 入口文件，需要导出一个函数
// 此函数接收一个 plop 对象，用于创建生成器任务
module.exports = plop => {
    plop.setGenerator('component', {
        description: 'create a component',
        prompts: [ // 命令行交互问题
            {
                type: 'input', // 方式 输入
                name: 'name',
                message: 'component name',
                default: 'MyComponent' // 默认值
            }
        ],
        actions: [ // 完成命令行交互过后执行的动作
            {
                type: 'add', // 添加新文件
                path: 'src/components/{{name}}/index.vue', // 双花括弧插值命令行中的 name 取值
                templateFile: 'plop-templates/component.hbs'
            }
        ]
    })
}
```
* 项目根目录下创建 `Handlebars` 模板文件 '/plop-templates/component.hbs'
```handlebars
<template>
  <div class="{{name}}">
    组件 {{name}}
  </div>
</template>

<script>
  export default {
    name: '{{name}}',
    data() {
      return {}
    },
    methods: {},
  }
</script>

<style scoped>
  .{{name}} {

  }
</style>
```
## 使用
* 命令行执行 `plop` 的 `cli` 命令，按提示输入待创建组件名称；
```sh
yarn plop component
```
![plop执行创建](plop执行创建.png)
![创建结束](创建结束.png)
![生成文件](生成文件.png)
![结果使用](结果使用.png)

* 也可在 `package.json` 中配置 `scripts` 命令，然后执行 `yarn plop` 同样可创建文件；
```json
"scripts": {
  "plop": "plop"
}
```
# 总结
`Plop` 是一个小而美的脚手架工具，可以轻松帮助我们按指定模板创建目标文件；

更多详细操作请查阅官网 [https://plopjs.com/documentation/](https://plopjs.com/documentation/)
