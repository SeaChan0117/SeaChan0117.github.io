---
title: Hexo 图片插入插件的使用
date: 2021-06-19 20:47:26
categories: 
- Hexo
tags:
- Hexo
---
都说 Hexo 是 markdown 语法，插入图片不也用语法插入就OK了么，但是我在插入图片时，遇到了问题。

我使用过 github 路径，码云路径，都有问题；具体可自行尝试，本文主要简述插件的使用。
哦，对了，我还使用过把图片转为 base64 编码的数据用于插入的，问题是解决了，新的麻烦又来了，文章要插入大量图片时，真的很烦QAQ。<!--more-->

# hexo-asset-image
使用 hexo-asset-image 插件为 hexo 博客添加插入图片，步骤：

* hexo 项目安装该依赖，yarn add hexo-asset-image；
* 修改配置文件 _config.yml
```yaml
post_asset_folder: true

url: https://xxxxxx.github.io  # 自己项目的 git 仓库
```
* 将需要引用的图片存放在与文章名同名的文件夹中
```
_posts
    -image-test
        -example.png
    -image-test.md
```
* 在 markdown 文档中直接使用模板语法插入图片
```markdown
{% asset_img example.png This is pic title %}
```
* 使用 markdown 语法插入图片
```markdown
![图片](example.png)
```

以上方法启动项目后，发现图片并没有加载成功，查看元素后发现路径中有 .io/ ，是路径处理问题，在 hexo-asset-image 源码中打印日志，找到了出错的地方。

解决方法就是自己重新搞一个 npm 仓库，把 hexo-asset-image 的问题修改后发布上 npm，用于拉取使用，目前已修改发布，包名是 hexo-asset-image-cs；使用步骤和上边一致，
只需要在第一步中把依赖拉取改为 hexo-asset-image-cs 即可。

**模板语法插入测试**
{% asset_img 2021-06-19_210650.png This is pic title %}

**markdown 语法插入测试**
![test](2021-06-19_210650.png)

# 声明
[hexo-asset-image-cs](https://github.com/SeaChan0117/hexo-asset-image-cs) 不是自己开发，
只是针对 [hexo-asset-image](https://github.com/xcodebuild/hexo-asset-image) 改了一下支持现在的 Hexo，自己只是搬运工。
