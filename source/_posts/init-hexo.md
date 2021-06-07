---
title: 通过 Hexo 和 Travis 构建一个属于自己的博客
---
　　**最近开启了大前端学习的道路，从事前端行业两年多了，想要一个自己的 Blog，不管是学习记录还是生活记录，都可以使用起来，而不是繁杂难选的三方平台，乘着学习过程中看到过的 Hexo＋Travis 的方式搭建自己的博客，我也兴致勃勃的想试试，此时你看到的就是搭建结果，这也是第一篇博客！**  
# 前置准备 #
　　流浪在大前端的今天怎么能没有 node 呢，node 是必备的=。=；  
　　在自己的 git 仓库新建一个 xxx.github.io 仓库名的仓库，注意此处的 xxx 为你的 git 账号名，我的就是 seachan0117,细心的你可能已经发现，就是浏览器域名中的前一部分，我新建的仓库就是 SeaChan0117.github.io；  
此时通过浏览器尝试访问 seachan0117.github.io，页面报404，原因是我们项目还是空的，将代码clone到本地，新建一个 index.html，其中可简单填写部分内容测试，提交至远程仓库后，再次访问 seachan0117.github.io，我们的页面就能访问了。 
其实到这我们的 Blog 就算基本搭建完成，自己可以项目中开发自己的 Blog，但是单纯手工以代码形式去编写 Blog 似乎太累了，也不利于维护，这时就需要用到 Hexo；   
*到这我就想，如果是其他项目的编译文件，是静态页面的情况下，如果放到该目录应该也是OK的，当然会涉及一些路径的问题，不过思路到这后续就可以尝试啦。*  
# Hexo是什么？ #
　　**A fast, simple & powerful blog framework**，官方简洁明了的对 hexo 做了说明。  
Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页；Hexo 提供了脚手架，帮助我们快速开始属于自己的 Blog 开发；  
## Hexo安装 ##
　　此处全局安装 hexo-cli；  
   ` yarn add globle hexo-cli 或者 npm install -g hexo-cli`  

　　安装成功后，执行 hexo，命令行会提示相关操作选项，我们此时来初始化 Hexo 开发项目 **hexo init SeaChan0117.github.io**，会在 SeaChan0117.github.io 目录下为我们生成一份初始化的 Hexo 开发项目，后续在其基础上使用 Markdown 语法开发我们的 Blog 即可；
    `hexo init SeaChan0117.github.io`  

　　初始化完成后，启动项目，Hexo 默认 http://localhost:4000 ，打开网址即可查看本地启动的 Hexo 项目；  
	`npm run server 或者 yarn server 或者 hexo s`
