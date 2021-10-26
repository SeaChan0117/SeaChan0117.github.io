---
title: 通过 Hexo 和 Travis 构建一个属于自己的博客
categories: Tools
tags: hexo
top: true
cover: https://tse2-mm.cn.bing.net/th/id/OIP-C._0Uk0LR-X6hfrBl7Jc1eDAHaEK?pid=ImgDet&rs=1
---
　　**最近开启了大前端学习的道路，从事前端行业两年多了，想要一个自己的 Blog，不管是学习记录还是生活记录，都可以使用起来，而不是繁杂难选的三方平台，乘着学习过程中看到过的 Hexo＋Travis 的方式搭建自己的博客，我也兴致勃勃的想试试，此时你看到的就是搭建结果，这也是第一篇博客！**  
# 前置准备 #
　　流浪在大前端的今天怎么能没有 node 呢，node 是必备的，大家自行安装即可=。=；  
<!--more-->
　　在自己的 git 仓库新建一个 xxx.github.io 仓库名的仓库，注意此处的 xxx 为你的 git 账号名，我的就是 seachan0117,细心的你可能已经发现，就是浏览器域名中的前一部分，我新建的仓库就是 SeaChan0117.github.io；  
此时通过浏览器尝试访问 seachan0117.github.io，页面报404，原因是我们项目还是空的，将代码clone到本地，新建一个 index.html，其中可简单填写部分内容测试，提交至远程仓库后，再次访问 seachan0117.github.io，我们的页面就能访问了。 
其实到这我们的 Blog 就算基本搭建完成，自己可以项目中开发自己的 Blog，但是单纯手工以代码形式去编写 Blog 似乎太累了，也不利于维护，这时就需要用到 Hexo；   
*到这我就想，如果是其他项目的编译文件，是静态页面的情况下，如果放到该目录应该也是OK的，当然会涉及一些路径的问题，不过思路到这后续就可以尝试啦。*  
# Hexo 是什么？ #
　　**A fast, simple & powerful blog framework**，官方简洁明了的对 hexo 做了说明。  
Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页；Hexo 提供了脚手架，帮助我们快速开始属于自己的 Blog 开发；  
## Hexo 安装 ##
此处全局安装 hexo-cli；  
` yarn add globle hexo-cli`  
或者  
`npm install -g hexo-cli`

## Hexo 初始化 ##
安装成功后，执行 hexo，命令行会提示相关操作选项，我们此时来初始化 Hexo 开发项目 **hexo init SeaChan0117.github.io**，会在 SeaChan0117.github.io 目录下为我们生成一份初始化的 Hexo 开发项目，后续在其基础上使用 Markdown 语法开发我们的 Blog 即可；
`hexo init SeaChan0117.github.io`  

初始化完成后，启动项目，Hexo 默认 http://localhost:4000 ，打开网址即可查看本地启动的 Hexo 项目；  
`yarn server`  
或者  
`npm run server`  
或者  
`hexo s`  
此时，Hexo 初始化项目就跑起来了，浏览器打开 http://localhost:4000 即可看到 Hexo 博客，是不是很简单呢。 根据界面上展示的内容在代码中查找，能知道页面的 .md 文档路径是 source\\_posts\下，尝试修改后，刷新界面验证自己的想法吧。  

我们可以使用 Hexo 为我们提供的命令生成新页面，放在 source\\_posts\ 下：  
`hexo new first-blog`  
执行上述命令后会新增 first-blog.md 的文件，并初始化了标题和日期  

那么问题来了，我们之前说了把项目名配置为 seachan0117.github.io，就是为了使用 github pages 功能，能浏览器直接访问 seachan0117.github.io 静态页面，怎么把 Hexo 项目在 seachan0117.github.io 访问呢？Hexo 官方为我们提供了一个解决方案 **Travis CI**。

# Travis CI 将 Hexo 部署 GitHub Pages
　　[Travis CI](https://www.travis-ci.org/) 提供的是持续集成服务（Continuous Integration，简称 CI）。它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。  
## Travis CI 使用准备
> 绑定 github 账号；  
> github 账号下有项目；  
> 项目中有可运行代码；  
> 该项目还包含有构建或者测试脚本；  

如果以上条件都满足，你就可以开始使用 Travis CI 了。

## Travis CI 使用流程
github 账号绑定登录后，进行授权， Travis CI 会列出你在 github 上的项目列表；  
在 github 的设置里 [https://github.com/settings/tokens](https://github.com/settings/tokens)，生成一个 Token，此处我的 Token 名是 **TOKEN**，下面的选项只勾选 repo，点击按钮生成 Token；  
复制 Token 值，以备后续使用；
Travis CI 中选中左侧栏对应的项目仓库，此处我是  SeaChan0117 / SeaChan0117.github.io，右边点击 More options 中的 settings；
在 settings 中的 Environment Variables 项填入我们上边复制的 Token，注意 Name 和 Value 对应，然后点击行后的 Add 按钮即可；
在本地项目 SeaChan0117.github.io 根目录下新建一个名为`.travis.yml`的文件，将以下内容复制进去，注意 github-token 项是你自己在 github 上生成的 Token 键名;
```yaml
sudo: false
language: node_js
node_js:
  - 12 # use nodejs v12 LTS
cache: npm
install:
  - yarn
branches:
  only:
    - master # build master branch only
script:
  - hexo clean # clean static files
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```
将 .travis.yml 文件提交至远程仓库，此时 Travis CI 上就会自动开启一个黑盒为启动部署，它会将生成的文件推送到同一 repository 下的 gh-pages 分支下；
当查看部署状态为 passed 后，回到 github 上，对应仓库的 settings 下，左侧栏选择 Pages，右边 Source 选择 gh-pages；
稍等片刻，再次浏览器中访问 [SeaChan0117.github.io](https://seachan0117.github.io)；
Bingo！！！

## 手动部署 ##
Hexo 提供了快速方便的一键部署功能，你只需一条命令就能将网站部署到服务器上；  
`hexo deploy`  
在开始之前，您必须先在 _config.yml 中修改参数，一个正确的部署配置中至少要有 type 参数，例如：
```yaml
deploy:
  type: git
```
您可同时使用多个 deployer，Hexo 会依照顺序执行每个 deployer；   
```yaml
deploy:
- type: git
  repo:
- type: heroku
  repo:
```
知道该配置项后，你还需要安装相关支持的依赖 **hexo-deployer-git**；  
`yarn add hexo-deployer-git`  
然后修改自己的配置文件，我的配置如下：
```yaml
deploy:
  type: 'git'
  repo: https://github.com/SeaChan0117/SeaChan0117.github.io
  branch: master
  message: "Sea Chan's blogs-deploy"
```
分别执行 `hexo clean` 和 `hexo deploy`，生成站点文件并推送到远程仓库，此时，再访问 seachan0117.github.io 即可，我们的 Blog 已经部署到服务上！  
接下来就可以开始书写自己的 Blog，那么问题又来了，每次编写 Blog 或其他文件后，都需要手动发布到服务上。

# 后序
现在你看到的是 Hexo 加上 theme-next 呈现的效果，比单纯的 Hexo 美化了很多，后序有时间再整理 theme-next 吧。
