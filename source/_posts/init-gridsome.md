---
title: gridsome 安裝
categories: Tools
tags: gridsome
cover: https://dev-to-uploads.s3.amazonaws.com/i/yeq5sadxv3265n70vody.jpeg
---

Gridsome 初始化安装，参照[官网](https://www.gridsome.cn/docs/#how-to-install) 

# 安装

全局安装 gridsome 的脚手架工具，```yarn global add @gridsome/cli```

# 创建项目

## sharp

官方给出在上一步安装脚手架后，可直接使用脚手架命令 ```gridsome create my-gridsome-site``` 创建一个名为 my-gridsome-site 的初始化项目，但是这样创建很难成功，因为 gridsome 依赖第三方的包 sharp，用于处理图片，它包含很多 C++ 文件，安装时需要编译，即需要 C++ 编译环境；sharp 还依赖一个 libvips 包，该包文件较大，在国内下载容易失败，即导致 sharp 难以安装成功；[sharp官网](https://sharp.pixelplumbing.com/install#chinese-mirror) 提供了一个中国镜像解决该问题；
<!--more-->
执行命令配置 sharp 镜像：

```sh
npm config set sharp_binary_host "https://npm.taobao.org/mirrors/sharp"
npm config set sharp_libvips_binary_host "https://npm.taobao.org/mirrors/sharp-libvips"
```

## C++ 编译环境

以上配置完之后，在执行初始化项目时会安装 sharp，我们还差一个 C++ 编译环境；

安装 node-gyp

```sh
yarn global add node-gyp
```

node-gyp 安装后，还需要安装 node-gyp 相关的使用套件，参考[官方库文档](https://github.com/nodejs/node-gyp#on-windows) ，此处我本地以 Windows 为例；

### [安装 Python](https://docs.python.org/3/using/windows.html#the-microsoft-store-package)

### 安装 windows-build-tools

```sh
npm install --global --production windows-build-tools
```

安装 windows-build-tools 过程安装失败，查了资料，要使用 32 位的 node.js，故重新安装 node，再执行上边命令安装 windows-build-tools 即可；

## 创建

执行 ```gridsome create my-gridsome-site``` 创建即可；

